# PR0301: Despliegue automatizado de servidor web mediante User Data y metadatos IMDSv2

### PASO:1. Creación del Grupo de Seguridad (Security Group) <br>

Antes de lanzar el cómputo, se deben definir las políticas de filtrado de tráfico.

1. En la consola de AWS, acceder a EC2 > Network & Security > Security Groups.

2. Pulsar en Create security group y rellenar los datos:

* Security group name: sg-servidor-web-asir

* Description: Permitir acceso HTTP publico y SSH desde el aula

* VPC: Seleccionar la VPC por defecto del laboratorio.
![alt text](image-2.png)

![alt text](image.png)
3. Configurar las reglas de entrada (Inbound rules):

![alt text](image-1.png)
![alt text](image-3.png)

1. Dejar las reglas de salida (Outbound rules) por defecto (todo el tráfico permitido hacia 0.0.0.0/0) para que la máquina pueda descargar paquetes del repositorio oficial de AWS.
![alt text](image-4.png)

2. Guardar pulsando en Create security group.

### Paso 2: Análisis del script de User Data
El siguiente script se inyectará en la instancia durante el asistente de lanzamiento. <br> 

* Creamos el Script
![alt text](image-5.png)


### Paso 3: Lanzamiento de la Instancia EC2

1. Ir a EC2 > Instances > Launch Instances.

2. Name and tags: Asignar el nombre web-server-{iniciales}-01.
![alt text](image-6.png)

3. Application and OS Images (Amazon Machine Image): Seleccionar Amazon Linux 2023 AMI (arquitectura x86_64).
![alt text](image-7.png)

4. Instance type: Seleccionar t2.micro o t3.micro
![alt text](image-8.png)

5. Key pair (login): Crear una clave personal .pem para acceder por terminal.
![alt text](image-9.png)

6. Network settings:
    * Pulsar en Edit.
    * VPC: VPC por defecto.

    * Auto-assign public IP: asegurarse de que esté en Enable.

    * Firewall (security groups): seleccionar Select existing security group y marcar sg-servidor-web-asir.
    ![alt text](image-10.png)

7. Advanced details:
    * Desplegar la sección inferior.
    * Metadata version: comprobar que esté seleccionado V2 only (token required) o V1 and V2 (token optional). Para forzar buenas prácticas, fijarlo en V2 only.
    ![alt text](image-11.png)
    
    * Localizar el cuadro de texto User data al final de la página y pegar el script completo del Paso 2. <br>
    
    ![alt text](image-12.png)
    Puedes Descargar el scrip que habiamos hecho y subirlo
    ![alt text](image-14.png)
    ![alt text](image-13.png)

8. Pulsar en Launch instance.
![alt text](image-15.png)

### Paso 4: Verificación y resolución de problemas

1. Comprobación vía navegador web

    1. En la lista de instancias, esperar a que el estado pase a Running y los dos chequeos de estado indiquen 2/2 checks passed.
    ![alt text](image-19.png)
    ![alt text](image-17.png)
    ![alt text](image-18.png)

    2. Copiar la Public IPv4 address o la Public IPv4 DNS.
    ![alt text](image-20.png)

    3. Abrir una pestaña en el navegador y acceder por HTTP (Asegurate de que el navegador no redirija automáticamente a https://, ya que el puerto 443 no está configurado).

    ![alt text](<WhatsApp Image 2026-10-16 at 13.21.34.jpeg>)

2. Inspección del log del sistema por SSH
    * Modifcamos las reglas para podernos conectar desde CloudShell por que antes solo nos podiamos conecatrde desde la red del instituto
    ![alt text](image-24.png)

    * Modifcamos los permisos de las claves para que no sea accesible para otros
    ![alt text](image-23.png)

    Para verificar el comportamiento de ```cloud-init``` y cómo se procesó el script:

    1. Conectarse a la máquina mediante SSH:
    ```  ssh -i "clave_privada.pem" ec2-user@<TU_IP_PUBLICA>```
    
    ![alt text](image-21.png)
    ![alt text](image-25.png)

    2. Inspeccionar el registro de salida de arranque:
    ```   sudo cat /var/log/cloud-init-output.log```
    
    ![alt text](image-27.png)

    3. Comprobar manualmente la consulta a IMDSv2 desde la propia terminal:

   ``` TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 60") ``` <br>

    ![alt text](image-16.png)
    
    * Este es lo que nos devuelve
    ![alt text](image-29.png)
