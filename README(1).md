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

![Server Manager](img1_server_manager.png)

### 2 – Colección RemoteApp
Resultado de Get-RDSessionCollection mostrando la colección LabRemoteApp activa.

![RD Session Collection](img2_rd_collection.png)

### 3 – Aplicación RemoteApp publicada
Resultado de Get-RDRemoteApp mostrando "Mi Pagina IIS" publicada con IE apuntando a http://localhost:8080.

![RemoteApp Publicada](img3_remoteapp.png)

### 4 – Página IIS personalizada
Página web en http://localhost:8080 con diseño oscuro y badges de ITLA y Seguridad de Redes.

![Página IIS](img4_iis_page.png)

### 5 – Portal RD Web Access (desde servidor)
Portal RD Web Access mostrando la app "Mi Pagina IIS" disponible.

![RD Web Portal](img5_rdweb_portal.png)

### 6 – Login al portal desde PC física
Acceso a https://10.0.0.21/RDWeb con credenciales LAB\Administrator desde la PC host.

![RD Web Login](img6_rdweb_login.png)

### 7 – Página IIS via RemoteApp (Método 1)
Página IIS abierta a través del portal RD Web Access desde la PC física.

![RemoteApp Funcionando](img7_remoteapp_page.png)

### 8 – Conexión RDP directa con mstsc (Método 2)
Ventana mstsc conectándose a WIN-P06OPNTM9N5.lab.local con credenciales de dominio.

![Conexión mstsc](img8_mstsc.png)

### 9 – Portal RD Web Access desde PC física
Vista del portal desde el navegador del cliente mostrando la app disponible.

![RD Web desde cliente](img9_rdweb_client.jpeg)

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
