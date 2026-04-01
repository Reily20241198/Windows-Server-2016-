# Configuración de RDP RemoteApp y RD Web Access en Windows Server 2019

**Estudiante:** Reily Rosario  
**ID:** 20241198  
**Institución:** ITLA – Instituto Tecnológico de las Américas  
**Materia:** Seguridad de Redes  
**Fecha:** 1 de abril de 2026  

---

## Objetivo

Configurar el servicio de **RDP RemoteApp** y **RD Web Access** en Windows Server 2019, publicar una página web personalizada de IIS como aplicación remota, y demostrar el acceso desde un cliente mediante los dos métodos disponibles: portal web y conexión RDP directa.

---

## Topología

| Componente | Detalle |
|---|---|
| Servidor | Windows Server 2019 (VM en VMware Workstation) |
| Hostname | WIN-P06OPNTM9N5 |
| Dominio | lab.local |
| IP del servidor | 10.0.0.21 |
| Máscara de subred | 255.255.255.0 |
| Gateway | 10.0.0.1 |
| Cliente | PC física host (Windows 11) |
| IP del cliente | 10.0.0.4 |
| Puerto RDP | 3389 |
| Puerto IIS | 8080 |
| Puerto RD Web Access | 443 (HTTPS) |

```
PC Host (10.0.0.4)
        |
        | RDP / HTTPS
        |
Windows Server 2019 (10.0.0.21)
├── Active Directory (lab.local)
├── Remote Desktop Services
│   ├── RD Session Host
│   ├── RD Connection Broker
│   └── RD Web Access (https://10.0.0.21/RDWeb)
├── IIS (http://10.0.0.21:8080)
└── RemoteApp: "Mi Pagina IIS"
```

---

## Requisitos previos

- Windows Server 2019 instalado en VMware Workstation
- Red configurada en modo Bridged para conectividad con el host
- Conectividad verificada con ping entre cliente y servidor

---

## Configuraciones utilizadas

### 1. Instalación de roles

```powershell
Install-WindowsFeature -Name RDS-RD-Server, RDS-Web-Access, RDS-Connection-Broker, Web-Server -IncludeManagementTools -Restart
```

Roles instalados: AD DS, DNS, IIS, RD Session Host, RD Web Access, RD Connection Broker.

### 2. Configuración de Active Directory

```powershell
Install-ADDSForest `
    -DomainName "lab.local" `
    -DomainNetbiosName "LAB" `
    -InstallDns `
    -SafeModeAdministratorPassword (ConvertTo-SecureString "Admin123!" -AsPlainText -Force) `
    -Force
```

### 3. Deployment de Remote Desktop Services

```powershell
Enable-PSRemoting -Force

New-RDSessionDeployment `
    -ConnectionBroker "WIN-P06OPNTM9N5.lab.local" `
    -WebAccessServer "WIN-P06OPNTM9N5.lab.local" `
    -SessionHost "WIN-P06OPNTM9N5.lab.local"
```

### 4. Creación de la colección RemoteApp

```powershell
New-RDSessionCollection `
    -CollectionName "LabRemoteApp" `
    -SessionHost "WIN-P06OPNTM9N5.lab.local" `
    -ConnectionBroker "WIN-P06OPNTM9N5.lab.local"
```

### 5. Página personalizada en IIS (puerto 8080)

```powershell
New-Item -Path "C:\inetpub\milab" -ItemType Directory -Force
Import-Module WebAdministration
New-Website -Name "MiLabRDP" -Port 8080 -PhysicalPath "C:\inetpub\milab" -Force
Start-Website -Name "MiLabRDP"
```

### 6. Publicación de la página IIS como RemoteApp

```powershell
New-RDRemoteApp `
    -CollectionName "LabRemoteApp" `
    -DisplayName "Mi Pagina IIS" `
    -FilePath "C:\Program Files\Internet Explorer\iexplore.exe" `
    -CommandLineSetting Require `
    -RequiredCommandLine "http://localhost:8080" `
    -ConnectionBroker "WIN-P06OPNTM9N5.lab.local"
```

### 7. Configuración del cliente (archivo hosts)

```
10.0.0.21    WIN-P06OPNTM9N5.lab.local
```

---

## Capturas de pantalla

### 1 – Roles instalados en Server Manager
Server Manager mostrando AD DS, DNS, IIS y Remote Desktop Services instalados correctamente.

<img width="1913" height="1044" alt="Screenshot_4" src="https://github.com/user-attachments/assets/9a2e18e3-9144-4dc5-b844-da0711fe9dd1" />


### 2 – Colección RemoteApp
Resultado de Get-RDSessionCollection mostrando la colección LabRemoteApp activa.

<img width="726" height="134" alt="Screenshot_5" src="https://github.com/user-attachments/assets/1d13ebaf-fb20-448c-b4a9-7b9c49253b0e" />


### 3 – Aplicación RemoteApp publicada
Resultado de Get-RDRemoteApp mostrando "Mi Pagina IIS" publicada con IE apuntando a http://localhost:8080.

<img width="915" height="117" alt="Screenshot_6" src="https://github.com/user-attachments/assets/408fb196-cfc6-4b52-b798-a22153a8f523" />


### 4 – Página IIS personalizada
Página web en http://localhost:8080 con diseño oscuro y badges de ITLA y Seguridad de Redes.

<img width="1429" height="771" alt="Screenshot_7" src="https://github.com/user-attachments/assets/d7daa13d-229f-4dd2-a00e-9157c98c2d23" />


### 5 – Portal RD Web Access (desde servidor)
Portal RD Web Access mostrando la app "Mi Pagina IIS" disponible.

<img width="1099" height="826" alt="Screenshot_8" src="https://github.com/user-attachments/assets/1e336fce-af7b-4eaf-b46a-97be627aa8e1" />


### 6 – Login al portal desde PC física
Acceso a https://10.0.0.21/RDWeb con credenciales LAB\Administrator desde la PC host.
<img width="1573" height="720" alt="Screenshot_9" src="https://github.com/user-attachments/assets/198bc4a8-6e65-48c4-bc5a-07a91226a98f" />



### 7 – Página IIS via RemoteApp (Método 1)
Página IIS abierta a través del portal RD Web Access desde la PC física.

<img width="1401" height="833" alt="Screenshot_10" src="https://github.com/user-attachments/assets/442aae13-2d76-4aa7-a531-e27059c3370b" />


### 8 – Conexión RDP directa con mstsc (Método 2)
Ventana mstsc conectándose a WIN-P06OPNTM9N5.lab.local con credenciales de dominio.

<img width="557" height="280" alt="Screenshot_11" src="https://github.com/user-attachments/assets/b6e972b1-844f-4647-aa36-87775169c0e8" />


### 9 – Portal RD Web Access desde PC física
Vista del portal desde el navegador del cliente mostrando la app disponible.

![WhatsApp Image 2026-04-01 at 3 36 13 PM](https://github.com/user-attachments/assets/afae7e43-3be5-4aa0-8058-6b6f4ddc99ca)


---

## Métodos de acceso demostrados

### Método 1 – RD Web Access (portal web)
1. Abrir navegador y navegar a `https://10.0.0.21/RDWeb`
2. Iniciar sesión con `LAB\Administrator`
3. Hacer clic en "Mi Pagina IIS"
4. Aceptar certificado y credenciales RDP
5. La página IIS se abre remotamente desde el servidor

### Método 2 – RDP directo con mstsc
1. Presionar `Windows + R` → escribir `mstsc`
2. Conectarse a `WIN-P06OPNTM9N5.lab.local`
3. Iniciar sesión con `LAB\Administrator`
4. Dentro de la sesión, abrir navegador en `http://localhost:8080`

---

## Conclusión

Se configuró exitosamente RDP RemoteApp en Windows Server 2019 con Active Directory, publicando la página personalizada de IIS como aplicación remota accesible desde un cliente externo mediante portal web (RD Web Access) y conexión RDP directa (mstsc). Ambos métodos fueron verificados y funcionaron correctamente.
