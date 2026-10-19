# **GUÍA DE DESPLIEGUE: SERVIDOR WEB NGINX CON DUCKDNS EN ORACLE CLOUD INFRASTRUCTURE (OCI)**

**Autores:**

* Javier Vargas Sánchez

* Andrés Fernando Quintana Quintero

**Materia:** Infraestructura Cloud / Oracle

**Fecha:** 16 de octubre de 2026

## **REQUISITOS PREVIOS**

* Cuenta activa en Oracle Cloud Infrastructure (OCI).

* Cuenta activa en [DuckDNS](https://www.duckdns.org/).

* Repositorio público en GitHub para alojar la guía e imágenes demostrativas.

---

## **PARTE 1: CONFIGURACIÓN DE GOBERNANZA, SEGURIDAD Y USUARIOS EN OCI**

### **1.1 Navegación al Módulo de Identidad y Seguridad**

1. Acceda a la consola de Oracle Cloud Infrastructure.

2. Ingrese al menú principal en la esquina superior izquierda y seleccione **Identity & Security**.

![Módulo Identity & Security en OCI](https://raw.githubusercontent.com/adanbeltran/Labs/main/Lab-Despliegue%20Web%20IaaS%20con%20OCI%20Compute/images/01_identity_security.png)

### **1.2 Verificación del Dominio de Identidad y Usuarios**

1. Dentro de **Identity**, seleccione la opción **Domains**.

2. Haga clic en el dominio `Default`.

3. Diríjase a **User management > Users** para verificar la existencia del usuario administrador (`aquintana@uniempresarial.edu.co`).

![Verificación de Dominio Default](https://raw.githubusercontent.com/adanbeltran/Labs/main/Lab-Despliegue%20Web%20IaaS%20con%20OCI%20Compute/images/02_identity_domains.png)

### **1.3 Creación del Grupo de Usuarios (`GrupoInfraestructura`)**

1. En la barra lateral del dominio `Default`, seleccione **Groups**.

2. Presione el botón **Create group**.

3. Complete los campos:

   * **Name:** `GrupoInfraestructura`

   * **Description:** Grupo para administración de recursos de infraestructura cloud y servicios web.

4. Asigne los usuarios correspondientes (incluyendo la cuenta de Andrés Quintana) al grupo.

![Creación del Grupo de Usuarios](https://raw.githubusercontent.com/adanbeltran/Labs/main/Lab-Despliegue%20Web%20IaaS%20con%20OCI%20Compute/images/03_create_group.png)

### **1.4 Configuración de Políticas y Permisos (`PermisosInfraestructura`)**

1. En el menú de navegación, seleccione **Identity & Security > Policies**.

2. Seleccione el compartimento correspondiente y presione **Create Policy**.

3. Configure la política:

   * **Name:** `PermisosInfraestructura`

   * **Description:** Permisos de gestión sobre recursos Compute y Virtual Cloud Networks.

4. Añada las siguientes declaraciones en el **Policy Builder**:

   ```
   Allow group GrupoInfraestructura to manage instance-family in compartment <nombre_compartimento>
   Allow group GrupoInfraestructura to manage virtual-network-family in compartment <nombre_compartimento>
   ```

![Creación de Política de Seguridad](https://raw.githubusercontent.com/adanbeltran/Labs/main/Lab-Despliegue%20Web%20IaaS%20con%20OCI%20Compute/images/04_create_policy.png)

### **1.5 Creación y Asignación del Usuario Estudiante (`estudiante-web`)**

1. Regrese a **Identity & Security > Domains > Default > Users**.

2. Haga clic en **Create user**.

3. Diligencie la información requerida:

   * **Username:** `estudiante-web`

   * **Email:** `aquintana@uniempresarial.edu.co`

4. Seleccione e integre el grupo `GrupoInfraestructura` en la asignación del usuario.

![Creación de Usuario Estudiante](https://raw.githubusercontent.com/adanbeltran/Labs/main/Lab-Despliegue%20Web%20IaaS%20con%20OCI%20Compute/images/05_create_user.png)

### **1.6 Ficha de Detalles del Usuario Creado**

1. Ingrese a la ficha individual del usuario `estudiante-web`.

2. Verifique la información general, correo electrónico de recuperación y pertenencia al grupo `GrupoInfraestructura`.

![Ficha de Usuario Registrado](https://raw.githubusercontent.com/adanbeltran/Labs/main/Lab-Despliegue%20Web%20IaaS%20con%20OCI%20Compute/images/06_user_details.png)

### **1.7 Activación de Cuenta, Configuración de MFA e Inicio de Sesión**

1. Revise el correo electrónico asociado (`aquintana@uniempresarial.edu.co`) y abra el mensaje de bienvenida enviado por Oracle Cloud.

2. Ingrese al enlace proporcionado para establecer la contraseña inicial de la cuenta `estudiante-web`.

3. Complete el registro obligatorio de **Autenticación Multifactor (MFA)** usando una aplicación como Oracle Mobile Authenticator o Google Authenticator.

4. Realice el primer inicio de sesión para confirmar el acceso correcto al panel de OCI.

![Activación de Cuenta y Perfil OCI](https://raw.githubusercontent.com/adanbeltran/Labs/main/Lab-Despliegue%20Web%20IaaS%20con%20OCI%20Compute/images/07_user_mfa_profile.png)

---

## **PARTE 2: CONFIGURACIÓN DE RED VIRTUAL (VCN) Y RECURSOS COMPUTE**

### **2.1 Creación de la Red Virtual (`red_infraestructura`)**

1. Diríjase a **Networking > Virtual Cloud Networks**.

2. Haga clic en **Create VCN**.

3. Defina los parámetros de red:

   * **Name:** `red_infraestructura`

   * **IPv4 CIDR Block:** `10.0.0.0/16`

![Creación de VCN](https://raw.githubusercontent.com/adanbeltran/Labs/main/Lab-Despliegue%20Web%20IaaS%20con%20OCI%20Compute/images/08_create_vcn.png)

### **2.2 Creación del Internet Gateway (`salida-internet`)**

1. Dentro del detalle de la VCN `red_infraestructura`, vaya al panel lateral y seleccione **Internet Gateways**.

2. Presione **Create Internet Gateway**.

3. Asigne el nombre `salida-internet` y guarde los cambios.

![Creación de Internet Gateway](https://raw.githubusercontent.com/adanbeltran/Labs/main/Lab-Despliegue%20Web%20IaaS%20con%20OCI%20Compute/images/09_create_igw.png)

### **2.3 Configuración de Tablas de Enrutamiento (Route Tables)**

1. En la VCN `red_infraestructura`, acceda a **Route Tables** y seleccione la **Default Route Table for red_infraestructura**.

2. Presione **Add Route Rules**.

3. Agregue la siguiente regla de salida a internet:

   * **Target Type:** `Internet Gateway`

   * **Destination CIDR Block:** `0.0.0.0/0`

   * **Target:** `salida-internet`

![Configuración de Reglas de Enrutamiento](https://raw.githubusercontent.com/adanbeltran/Labs/main/Lab-Despliegue%20Web%20IaaS%20con%20OCI%20Compute/images/10_route_tables.png)

### **2.4 Configuración de Subredes (`subnet-web`) y Listas de Seguridad**

1. En la VCN, seleccione **Subnets** y haga clic en **Create Subnet**:

   * **Name:** `subnet-web`

   * **Subnet Type:** Public Subnet

   * **IPv4 CIDR Block:** `10.0.0.0/24`

   * **Route Table:** `Default Route Table for red_infraestructura`

2. Diríjase a **Security Lists** y abra la **Default Security List for red_infraestructura**.

3. Haga clic en **Add Ingress Rules** para permitir el tráfico web HTTP:

   * **Source Type:** CIDR

   * **Source CIDR:** `0.0.0.0/0`

   * **IP Protocol:** TCP

   * **Destination Port Range:** `80`

![Creación de Subred Pública](https://raw.githubusercontent.com/adanbeltran/Labs/main/Lab-Despliegue%20Web%20IaaS%20con%20OCI%20Compute/images/11_create_subnet.png)

### **2.5 Creación de la Instancia Compute (`servidor-web`)**

1. Navegue a **Compute > Instances** y presione **Create instance**.

2. Configure los detalles de la máquina virtual:

   * **Name:** `servidor-web`

   * **Image:** Ubuntu (22.04 LTS o superior)

   * **Networking:**

     * **VCN:** `red_infraestructura`

     * **Subnet:** `subnet-web`

     * **Public IPv4 Address:** Assign a public IPv4 address

3. Descargue y guarde las claves SSH generadas para la conexión privada.

4. Presione **Create**.

![Creación de Instancia Compute](https://raw.githubusercontent.com/adanbeltran/Labs/main/Lab-Despliegue%20Web%20IaaS%20con%20OCI%20Compute/images/12_create_instance.png)

---

## **PARTE 3: INSTALACIÓN DE NGINX Y CONFIGURACIÓN DEL SERVIDOR WEB**

### **3.1 Conexión SSH e Instalación de Nginx**

1. Abra su terminal local y conéctese a la máquina virtual mediante SSH:

   ```bash
   ssh -i "ruta/a/tu_llave.key" ubuntu@<IP_PUBLICA_DE_LA_VM>
   ```

2. Actualice los repositorios del sistema e instale el servidor web Nginx:

   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install nginx -y
   ```

3. Habilite e inicie el servicio de Nginx:

   ```bash
   sudo systemctl enable --now nginx
   ```

4. Configure el firewall interno para abrir el puerto 80:

   ```bash
   sudo ufw allow 'Nginx HTTP'
   sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT
   sudo netfilter-persistent save
   ```

### **3.2 Creación del Directorio para el Sitio Web**

Cree el directorio raíz donde se ubicarán las secciones de la página web y asigne la propiedad del directorio al usuario actual:

```bash
sudo mkdir -p /var/www/mipagina
sudo chown -R $USER:$USER /var/www/mipagina
```

### **3.3 Configuración del Server Block en Nginx**

1. Cree el archivo de configuración para la nueva página en Nginx:

   ```bash
   sudo nano /etc/nginx/sites-available/mipagina
   ```

2. Añada el siguiente bloque de servidor (sustituya con su subdominio real de DuckDNS):

   ```nginx
   server {
       listen 80;
       server_name tu-subdominio.duckdns.org;

       root /var/www/mipagina;
       index index.html;

       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```

3. Active el sitio web mediante un enlace simbólico y verifique la sintaxis:

   ```bash
   sudo ln -s /etc/nginx/sites-available/mipagina /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl restart nginx
   ```

![Validación de Configuración Nginx](https://raw.githubusercontent.com/adanbeltran/Labs/main/Lab-Despliegue%20Web%20IaaS%20con%20OCI%20Compute/images/13_nginx_status.png)

---

## **PARTE 4: REGISTRO DE SUBDOMINIO EN DUCKDNS Y PRUEBA FINAL**

1. Ingrese al panel oficial de [DuckDNS](https://www.duckdns.org/) e inicie sesión.

2. Cree un subdominio propio en la plataforma (ejemplo: `tu-subdominio.duckdns.org`).

3. Ingrese la **IP pública** de la instancia Compute de OCI en el campo correspondiente y actualice el registro (**Update IP**).

4. Abra un navegador web externo y acceda a la dirección:

   `http://tu-subdominio.duckdns.org`

5. Verifique que la página web responda correctamente desde internet.

![Vinculación en DuckDNS y Verificación Web](https://raw.githubusercontent.com/adanbeltran/Labs/main/Lab-Despliegue%20Web%20IaaS%20con%20OCI%20Compute/images/14_duckdns_verify.png)

---

## **PARTE 5: PUBLICACIÓN EN GITHUB**

1. Abra una terminal en la carpeta local que contiene el archivo `.md` y las imágenes de las capturas.

2. Inicialice el repositorio Git y envíe los cambios a GitHub:

   ```bash
   git init
   git add .
   git commit -m "docs: guia de despliegue nginx en oci con duckdns por Javier Vargas y Andres Quintana"
   git branch -M main
   git remote add origin https://github.com/TU_USUARIO/NOMBRE_REPOSITORIO.git
   git push -u origin main
   ```

3. Compruebe que el repositorio sea público para asegurar la visibilidad de la guía y sus imágenes.