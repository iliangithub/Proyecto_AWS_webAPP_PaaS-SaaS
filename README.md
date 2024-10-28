# 0.0 Proyecto AWS webAPP2 PaaS & SaaS ("epsilon").

>[!IMPORTANT]
>Venimos de aquí:
>
>https://github.com/iliangithub/Proyecto_AWS_webAPP
>

En realidad, ya hemos terminado el proyecto "delta".

Este proyecto, va a pasar de ser "delta" a ser **"epsilon"**.

Ahora, vamos a volver a hacerlo pero modificando ciertas cosas. Vamos a cambiar la arquitectura de los servicios para AWS Cloud. Con la idea de mejorar la agilidad.

## 8.1 Escenario, nuevo proyecto "epsilon":

Digamos que estamos trabajando en un proyecto, en donde los servicios corren en máquinas físicas/virtuales o incluso pues en la nube, como las instancias EC2, y tenemos varios servicios, como aplicaciones web, de red, DNS, DHCP, bases de datos.

Necesitaríamos pues varios equipos distintos, uno para la parte de Cloud, un equipo de virtualización, uno de monitorización, administradores de sistemas, etc.

Entonces, hay muchas operaciones, y es complicado manejar, consume mucho tiempo y dinero.

Para ello, podemos utilizar la plataforma Cloud, pero en vez de IaaS, usaremos PaaS y SaaS. (Platform as a Service) y (Service as a Service).

Entonces, en caso de AWS, no vamos a ir con instancias corrientes, usaremos, algunos servicios de AWS y podríamos escribir como codigo nuestra infraestructura IaaC.

PaaS y SaaS son felxibles y fáciles de escalar.

Entonces, en vez de utilizar instancias EC2 normales para instalar nuestros servicios, utilizaremos Beanstalk.

> [!TIP]
> Elastic Beanstalk, un servicio de administración de aplicaciones en la nube que facilita el despliegue y gestión de aplicaciones web sin tener que preocuparse por la infraestructura subyacente. 
>

Beanstalk tendrá un balanceador de carga y autoescalado, almacenamiento S3 para almacenar los artefactos, además del servidor APP.

En cuanto al Backend, para las base de datos, utilizaremos instancias RDS, vamos a utilizar Elastic Cache en vez de MemCached y Active MQ, en vez de RabbitMQ, el Route 53 como DNS y Cloud Front para "content delivery network".

## 8.2 Objetivo:

- Necesitamos una infraestructura flexible.
- "no upfront cost", es decir, pagar por lo que usas, mientas lo usas.
- IaaC
- PaaS
- SaaS

![image](https://github.com/user-attachments/assets/f450d367-0856-4c85-8753-a3f931c86a7b)

Entonces, el usuario accederá a nuestra URL, que será resuelta a un punto final desde Amazon Route 53.

El punto final será parte de Amazon CloudFront, una red de entrega de contenido, que almacenará en caché muchas cosas para servir a la audiencia global.

Desde allí, la solicitud será redirigida al balanceador de carga de aplicaciones, que es parte de tu Elastic Beanstalk. El balanceador de carga de aplicaciones enviará la solicitud a la instancia de EC2, que está en un grupo de escalado automático. Aquí es donde estará ejecutándose nuestro servicio de aplicación Tomcat, y todo esto será parte de Elastic Beanstalk.

También habrá alarmas de Amazon CloudWatch que monitorearán el grupo de escalado automático y escalarán hacia arriba o hacia abajo según sea necesario.

Habrá un bucket donde se almacenarán los artefactos, y podremos desplegar nuestro artefacto más reciente simplemente haciendo clic en un botón.

Por lo tanto, todo nuestro frontend será gestionado por Beanstalk. Para el backend, en lugar de RabbitMQ, estamos utilizando Amazon MQ.

En lugar de usar Memcached en la instancia de EC2, estamos usando ElastiCache.

Y en lugar de usar una base de datos que se ejecute en una instancia de EC2, vamos a utilizar Amazon RDS.

Así que el usuario accederá a un punto final.

Ese punto final será parte de Amazon CloudFront, que enviará la solicitud al balanceador de carga de aplicaciones en Beanstalk, el cual reenviará la solicitud a las instancias en el grupo de escalado automático.

Todo esto será monitoreado por Amazon CloudWatch, que tendrá alarmas. Los artefactos se almacenarán en el bucket de S3.

Para el backend,

Accederá a Amazon MQ, ElastiCache y Amazon RDS Service.

Nuevamente, te recomiendo pausar el video y observar este diseño una vez más.

Veamos ahora el flujo de ejecución.

Primero, obviamente iniciaremos sesión en nuestra cuenta de AWS. Crearemos un par de claves para nuestra instancia de Beanstalk, o Beanstalk lanzará una instancia de EC2, así que crearemos un par de claves, de modo que, en caso de que necesites iniciar sesión, puedas usar ese par de claves.

Crearemos un grupo de seguridad para los servicios de backend: ElastiCache, RDS y ActiveMQ.

Luego, crearemos RDS,

ElastiCache y Amazon ActiveMQ.

Después, crearemos el entorno de Elastic Beanstalk.

Luego, actualizaremos nuestro grupo de seguridad del backend para permitir tráfico desde el grupo de seguridad de Beanstalk, de modo que cuando Beanstalk sea creado, también creará un grupo de seguridad para su instancia de EC2 y también para el balanceador de carga.

Permitiremos tráfico desde el grupo de seguridad de la instancia de Beanstalk para acceder a nuestros servicios de backend, que están en el grupo de seguridad del backend.

Estamos colocando todos nuestros servicios de backend en un grupo de seguridad, y necesitarán interactuar entre sí, por lo que también actualizaremos el grupo de seguridad del backend para permitir el tráfico interno.

Para este momento, nuestros servicios de backend también estarán funcionando. RDS estará funcionando, y necesitaremos inicializar nuestra base de datos de RDS.

Así que lanzaremos una instancia de EC2.

Y desde allí haremos un inicio de sesión de MySQL en nuestro RDS y inicializaremos nuestra base de datos.

Si seguiste nuestro proyecto anterior, sabrás que nuestra aplicación de perfil devuelve una página en /login, por lo que necesitamos cambiar la verificación de estado en Beanstalk.

De modo que cuando despleguemos nuestro artefacto, debería hacer una verificación de estado en /login.

Y también agregaremos un listener HTTPS en el puerto 443 a nuestro balanceador de carga elástico (ELB), que también será creado automáticamente por Beanstalk y será parte de tu entorno de Beanstalk.

Después, construiremos los artefactos a partir de nuestro código fuente con la información del backend.

Para este momento, deberíamos tener el punto final de RDS, el punto final de Amazon MQ y el punto final de ElastiCache. Ingresaremos esta información en nuestro archivo de propiedades de la aplicación y construiremos el artefacto.

Luego desplegaremos el artefacto en el entorno de Beanstalk.

Y crearemos una red de entrega de contenido utilizando Amazon CloudFront con un certificado SSL, por supuesto, para una conexión HTTPS.

Una vez que tengamos esto listo, podemos actualizar nuestro balanceador de carga y el punto final en GoDaddy, o también podemos hacerlo en Amazon Route 53, en zonas DNS públicas.

Una vez que todo esto esté listo, finalmente lo probaremos desde la URL.

OK, ahora hagamos que esto suceda.

Así que si has terminado de ver la introducción, únete a mí en la consola de AWS.

## 8.3 Crear las pares-clave y Grupos de Seguridad.

A partir de ahora, voy a poner a modo de resumen; qué es, y cómo lo hecho, sin tantas capturas.

### 8.3.1  Grupos de Seguridad.

En EC2 también.

> [!IMPORTANT]
> Si por casualidad, hemos borrado todos los grupos de seguridad, incluso el "default", para crearlo de nuevo, simplemente en acciones le damos a "crear grupo por defecto".
> 

- Name: `epsilon-rearch-backend-SG`
- Descripción: `epsilon-rearch-backend-SG`
- Inbound: NADA

Y LUEGO VAMOS A VOLVER A AÑADIRLE OTRA REGLA, (para permitir al acceso desde Beanstalk EC2 instance )

Inbound Rule.
- Type: All traffic
- Source: Custom, `epsilon-rearch-backend-SG`
- Description: `Allow all traffic internally`

> [!IMPORTANT]
> Luego, cuando creemos el "Beanstalk" volveremos a editarlo también.
> Para añadirle otra regla.
>

### 8.3.2 Crear las pares-clave.

Sería conveniente, crear un par-clave, para nuestra instancia "beanstalk", no es necesario imprescindible, iniciar sesión en la instancia "Beanstalk".

Nos vamos a EC2 y creamos el par de clave:

- Name: `epsilon-bean-key`
- File Format: .pem

y la creamos. ![image](https://github.com/user-attachments/assets/3e9e16f8-f93e-4bc5-bff2-2fb19c2cd9c0)

## 8.4 RDS
Vamos a la barra de búsqueda y buscamos RDS. Podríamos directante crear nuestra instancia DB, en el apartado del dashboard, "DB instances".

![image](https://github.com/user-attachments/assets/2238182a-bb68-4d20-8c53-ccd487595d36)

**No lo vamos a hacer. En la realidad, habrá situaciones en las que tenemos que cambiar algún parámetro de nuestra Base de Datos, RDS no te proporciona ningún acceso del estilo SSH, donde entramos y configuramos. Si queremos cambiar los parámetros de nuestra Base de datos, hay un concepto llamado "parameter group" en RDS.**

Entonces, vamos a crear el "parameter group", para así cuando lo seleccionemos, cuando queramos, podamos hacer cambios a la Base de Datos y se verá reflejado pues en la instancia RDS..
Antes de crear el RDS, vamos a crear el :
- parameter group.
- subnet group.

### 8.4.1 Creación Parameter Group.

En el Dashboard, buscamos "Parameter Group" y le damos a "create".

![image](https://github.com/user-attachments/assets/98ed9cf6-eef2-4e67-b779-fd8986283851)

- Parameter Group Name: `epsilon-rds-parametgrp`
- Description: `epsilon-rds-parametgrp`
- Engine Type: `MySQL Community.`
- Parameter Group Family: `MySQL8.0`
- Type: `DB Parameter Group`

Y lo creamos. Si le damos al parameter group, podemos ver todo lo que podemos modificar.

### 8.4.2 Creación Subnet Group.

Ahora es turno de los "subnet group". Es un grupo de subredes en una VPC (Virtual Private Cloud). Básicamente, es una red que está divida en otras redes más pequeñas llamadas pues subredes.

Cuando lancemos nuestra instancia RDS tenemos que seleccionar el "subnet group".

>[!TIP]
>Cuando seleccionamos diferentes subredes en casos de producción, necesitamos crear pues nuestra propia, VPC en un "subnetgroup" para el RDS y seleccionamos pues esas subredes a la hora de lanzar la instancia RDS.
>

Vamos a crear el subnet group.
Create DB subnet group:
- Nombre: `epsilon-rds-rearch-subgrp`
- Descripción: `epsilon-rds-rearch-subgrp`
- VPC: `aquí pues seleccionamos la VPC que hayamos creado, pero en nuestro caso, la default.`

Add Subnets:
- Availability Zone: Seleccionamos todas las zonas `us-east1a ; us-east1b ; ... us-east-1f`
- Subnets, seleccionamos todas las subredes de cada zona `us-east-1a --> subnet-055a1... ; us-east-1f --> subnet-01ed...`

Algunas zonas pueden tener varias "subnets".

>[!TIP]
>En algunos casos de producción, la gente crea una "subnet" diferente para la Base de datos. Y entonces, puedes seleccionar esas subredes (las de antes), y llamarlo "subnet group". Se hacer por motivos de seguridad y tener más control sobre la Base de datos.
>

Y creamos. En esta subred, vamos a crear/correr nuestra instancia de Base de Datos.

### 8.4.3 RDS creación de la instancia.

Nos vamos al Dashboard y la creamos.

Crear base de datos.

Elegir un método de creación de base de datos:
- Seleccionamos `standard create`

Opciones del motor:
- Tipo de motor: `MySQL`.
- Edición: Comunidad de MySQL (seleccionado, no puedes cambiarlo)
- `Mostrar solo versiones compatibles con las escrituras optimizadas de Amazon RDS` y `Mostrar solo las versiones compatibles con el clúster de base de datos multi-AZ` **deshabilitados**.
- Engine version: `MySQL 8.0.39`.
- `Enable RDS extended support` **deshabilitado**

Templates:
- Templates: `Free Tier`.

Nos saltamos "Availability and Durability" y en Settings:
- DB instance identifier: `epsilon-rds-rearch`
- username: `admin`
- Credentials management: `self managed`
- **Marcamos** auto generate password.

Instance configuration:
- DB instance class: `Show instance classes that support Amazon RDS Optimized Writes` y `Include previous generation classes`, **deshabilitado**; `db.t4g-micro`
  
En storage: 
- storage type: `general purpose SSD (gp3)` y `20 GB`
- **DESPLEGAMOS EL:** Escalado automático de almacenamiento **DESMARCAMOS "ENABLE STORAGE AUTOSCALING"**.

En conectivity:
- No se conecte a un recurso informático EC2.
- Nube privada virtual (VPC): `La default`.
- DB subnet group: `epsilon-rds-rearch-subgrp`.
- public access "No".
- VPC security Group (firewall): `choose existing`
- Existing VPC security groups: `epsilon-rearch-backend-sg` **y quitamos la default.**
- Availability zone `no preference`

El monitoring/supervisión deshabilitado.

Y por último, en "Additional configuration"
Initial database name: `accounts`.
DB parameter group: `epsilon-rds-parametgrp`

Copia de seguridad:
- **Backup deshabilitado**
- encryption `habilitado`

Mantenimiento:

- Habilitar actualización automática de versiones secundarias (**habilitado**).
- Periodo de mantenimiento: **Sin preferencia**
  
Protección contra eliminación
- Habilitar la protección contra la eliminación (**DESHABILITADO**)

> [!TIP]
>(Log exports, debería de estar habilitado, pero como no hemos habilitado el monitoring pues tenemos que dejarlo también deshabilitado, pues no nos sirve, básicamente va a los logs de CloudWatch).
>

Y lo creamos.

>[!IMPORTANT]
>Nos aparecerá un PopUp, diciendo de crear un ElastiCache CLuster o un RDS Proxy, vamos a cerrarlo.
>
>Arriba del todo nos aparecerá "VIEW CREDENTIALS DETAILS", pue shemos generado una contraseña del usuario de la Base de Datos, la necesitamos pues guardar/copiar.
>```
>eJmCwVwQjYRY22wRtftv
>```
>luego borraré la cuenta.

>[!IMPORTANT]
> En caso de haber perdido la contraseña, seleccionamos la BD y le damos a "Modify" y así generamos una nueva, pero claro, no podemos pues recuperar la antigua.
>

![image](https://github.com/user-attachments/assets/fc8504e6-55f1-47bf-aa47-ef25979215af)

## 8.5 ElastiCache.

Como antes, lo buscamos en la barra de búsqueda.

Y la creación del ElastiCache, es muy similar al RDS. Tenemos que crear:
- parameter group.
- subnetgroup.

### 8.5.1 Parameter Group, creación.

- Nombre: `epsilon-rearch-cache-paragrp`
- Descripción: `epsilon-rearch-cache paragrp`
- Family: `memcached1.6`

Y lo creamos.

### 8.5.2 Subnet Group, creación.

- Nombre: `epsilon-rearch-cache-subgrp`
- Descripción: `epsilon-rearch-cache-subgrp`
- VPC ID: `la por defecto`

Todas las subnets están seleccionadas. Lo creamos.

### 8.5.3 ElastiCache, creación.
Ahora, nos volvemos al Dashboard y "create cache", create memcached cache.

![image](https://github.com/user-attachments/assets/543038cc-34e1-4160-8bc9-1c71c42f9d6a)

> #### PASO 1. Configuración del clúster.
>- Design your own cache
>- Standard create
>
>Ubicación
>- AWS cloud.
>
>Información del clúster:
>- Name: `epsilon-rearch-cache`
>- Description: `epsilon-rearch-cache`.
>
>Cluster settings 
> - Engine version: `1.6.22`
> - port: `11211`
> - parameter group: `epsilon-reach-cache-paragrp`
> - node type: `cache.t2.micro`
> - number of nodes: 1
>
>Subnet group settings, (elija un grupo de subredes existente).
> - subnet groups: `epsilon-rearch-cache-subgrp`
>
>Ubicación de zonas de disponibilidad:
> - availability zone: "None preference".

> #### PASO 2. Configuración avanzada.
> Le damos a siguiente y llegamos a Advanced Settings:
> Segurity, (LE DAMOS A MANAGE/ADMINISTRAR):
> - Security Group: `epsilon-rearch-backend-sg`
> - Maintenance: `No preference.`

Y lo creamos.

## 8.6 Amazon MQ.
Lo mismo, buscamos en la barra de navegación "Amazon MQ", le damos a "Get Started".
- Broker engine: Rabbit MQ
- Deployment Mode: Simple-instance broker

Detalles:
- Broker Name: `epsilon-rearch-rabbitmq`
- Broker instance type: `mq.t3.micro`

Acceso a RabbitMQ:
- Username: `rabbit`
- Password: `BlueBunny983`

DESPLEGAMOS Additional Settings. 
- Broker engine version: 3.13

Configuración de agentes
- Crear una configuración nueva con los valores predeterminados

Monitoreo.
- CloudWatch **deshabilitado** (recordemos que esto en la vida real pues debería de ser así).

Redes y seguridad.
- Access type: private access.

Grupos de seguridad:
- Seleccionar grupos de seguridad existentes: `epsilon-rearch-backend-sg`

Lo creamos.

![image](https://github.com/user-attachments/assets/318ba244-b84f-4f45-bd5a-1d41322f03a9)

## 8.7 Inicializar la BD.
### 8.7.1 Crear la instancia.
Nos falta solamente, como tenemos la BD creada, nos falta coger el esquema de la BD y aplicarlo en nuestra instancia RDS.

Para ello, podemos hacer DOS cosas:
Hacerlo desde el MySQL Workbench CE:

![image](https://github.com/user-attachments/assets/192ddc06-1b10-4bed-a876-0f2b8f8cf991)

o también podemos pues crear una instancia EC2, descargar MySQL y utilizarlo para importar la BD.
Vamos a hacerlo de esa forma.

Voy al buscador de arriba y escribo EC2 y creo una instancia:
(NO HAY QUE HACER UNA INSTANCIA CURRADA, LA VAMOS A BORRAR, LA CREAMOS SOLO PARA USAR LOS COMANDOS SQL)

Name: `mysqli_cliente`
AMI: `Ubuntu`
Key-pair: `el que creamos antes del bean`

![image](https://github.com/user-attachments/assets/c739db1f-8810-4c9a-8b46-2e550fec87ad)

![image](https://github.com/user-attachments/assets/a3b3e019-7a00-4bca-aa9a-51bc6c34c123)

la creamos, no tocamos nada más.

### 8.7.2 Modificar el grupo de seguridad.

Ahora, necesitamos dos cosas:
- Los datos de la Base de datos RDS endpoint, el usuario y contraseña. Entonces, la información del RDS.
- Permitir que el grupo de seguridad de esta insancia permita, comunicarse con el grupo de seguridad del RDS.

Aunque esté corriendo, vamos a copiar el ID del `mysql-client-sg`:

![image](https://github.com/user-attachments/assets/523d073b-3f89-4f10-8678-85196ee4e962)

vamos al backend-sg y creamos una regla de entrada:

MySQL /Aurora ; Custom ; sg-082ba0c0d241bf36b

Habilitado desde el origen pues del cliente MySQL, para eso hemos copiado el ID.

### 8.7.3 YA TENEMOS LOS DATOS RDS, DEL USER Y PASS... Vamos a conectarnos e inicializarla.

Nos conectamos por ssh, IP pública + clave:

```
ssh -i "epsilon-bean-key.pem" ubuntu@ec2-54-157-166-106.compute-1.amazonaws.com
```

```
sudo -i
```

```
apt update && apt install mysql-client git -y
```

```
git clone https://github.com/hkhcoder/vprofile-project.git
```
Y lo tienes descargado en el directorio en el que te encuentres.

Hacemos un cd.

```
cd vprofile-project/
```

```
git checkout awsrefactor
```

Necesitamos saber cual es el ID o el enlace para el RDS:

![image](https://github.com/user-attachments/assets/cc911f0d-6013-467d-b705-be15afc22d44)

```
mysql -h epsilon-rds-rearch.crmqiuq428z2.us-east-1.rds.amazonaws.com -u admin -peJmCwVwQjYRY22wRtftv accounts < src/main/resources/db_backup.sql
```

La constraseña viene en el apartado 8.4.3 al final del todo.

Si hacemos un `SHOW tables;`

![image](https://github.com/user-attachments/assets/492cb849-3b43-4ef9-afb6-a49a5003455a)

Si nos da este resultado, vamos a eliminar pues la instancia.
Todo esto lo hacemos, solo porque la base de datos RDS es privada, y necesitamos pues estar en la misma red.

## 8.8 Amazon Elastic Beanstalk.

Vamos a recordar por un momento en "delta", cuando teníamos la instancia TomCat.
- Creamos el grupo de seguridad y las par-claves.
- Lanzamos la instancia e instalamos TomCat.
- Creamos un "Target group", load balancer y todo bien, una AMI... etc, etc...

Entonces, **Cuando creamos un "beanstalk enviroment", seleccionamos TomCat y pues nos proporciona todo lo anterior.** Un Autoscaling group, AMI, S3 bucket for the artifact, CloudWatch monitoring Logs, y es muy fácil cambiar los ajustes cuando quieras y el despliegue es muy fácil.

Lo primero es crear roles IAM para Beanstalk.

### 8.8.1 IAM, crear rol.

En la barra de navegació buscamos IAM:

- Tipo de entidad de confianza: `Servicio de AWS`.
- Servicio o caso de uso: `EC2`
- Caso de uso: `EC2`

Agregar permisos:
- AdministratorAccess-AWSElasticBeanstalk
- AWSElasticBeanstalkCustomPlatformforEC2Role
- AWSElasticBeanstalkRoleSNS
- AWSElasticBeanstalkWebTier

- Name: `epsilon-rearch-beanstalk-role`
- Description: `epsilon-rearch-beanstalk-role`

![image](https://github.com/user-attachments/assets/deef5304-80af-41c3-a543-2833c4527d22)

### 8.8.2 Beanstalk, creación.
Buscamos en la barra de navegación, "Elastic BeanStalk".

Le damos a **Crear Aplicación**.

#### PASO 1. Configuración del entorno.

> 
> - Nivel de entorno: `Entorno de servidor web`
>
> Información de la aplicación:
> - Nombre:
>   ```
>   epsilon-rearch-beanapp
>   ```
>
>Información del entorno:
> - Nombre del entorno:
>  ```
>  Epsilon-rearch-beanapp-prod
>  ```
>  
>- Dominio:
>  ```
>  epsilonrearch
>  ```
>  (tiene que ser único)
>
> Plataforma
>- Tipo de plataforma: `Plataforma administrada.`
>- Plataforma: `TomCat`
>- Ramificación de la plataforma: `Tomcat 10 with Correto 21 running...`
>- Versión: `5.3.3`
>
>Código de aplicación
>- Aplicación de ejemplo
>
>Valores preestablecidos
>- Configuración personalizada
>  

#### PASO 2. Configuración del acceso al servicio.

>
>Acceso al Servicio:
>- Rol de servicio: `Crear y utilizar un nuevo rol de servicio`
>- Nombre del rol de servicio: "el que está por defecto" `aws-elasticbeanstalk-service-role`
>- EC2 key pair: `epsilon-bean-key`
>- EC2 perfil de instancia: `epsilon-rearch-beanstalk-role`
**EL ROL QUE CREAMOS JUSTO EN EL 8.8.1**

> [!WARNING]
> Primero que nada, abre una ventana nueva con el AWS, vete a IAM > Roles, busca si NO ESTÁ CREADO DE ANTES EL ROL: `aws-elasticbeanstalk-service-role` si está creado de antes, bórralo, y continúa.
> 

#### PASO 3. Configuración del acceso al servicio.

>VPC.
>- VPC: `la default`
>- Public IP address (**ACTIVATED**)
>- Elegimos todas las subredes de instancia.
>- Pero no marcamos ninguna en: "Elegir subredes de base de datos"
>- Añadimos etiqueta, `Project` `Epsilon`

#### PASO 4. Configuración del escalado y del tráfico de instancias - opcional

>en este apartado solo vamos a tocar:
>Grupo de escalado automático
>- Tipo de entorno: `Equilibrio de carga.`
>- Instancias: `2Mín.` `4Máx.`
>- Tipo de instancia: `t2.micro`

#### PASO 5. Configuración de actualizaciones, monitoreo y registros.
![image](https://github.com/user-attachments/assets/b4326a12-b3cb-47be-93f3-8c5b732862a5)

![image](https://github.com/user-attachments/assets/0a789eed-b5a3-4c7c-b758-cce200d3a290)

![image](https://github.com/user-attachments/assets/d68a10f8-4214-4d93-b5fe-113d388e0652)

y ya en teoría está todo, creamos el beanstalk y si vemos el enlace:

![image](https://github.com/user-attachments/assets/5220420e-47fe-40a1-b599-b2382b69c40a)

>[!IMPORTANT]
>Me da error:
>
>![image](https://github.com/user-attachments/assets/78fd74ca-6229-46d8-b0bd-bd51fd8a580a)
>
>Ahora, estoy viendo este error:
>![image](https://github.com/user-attachments/assets/43bdf66b-e7fd-48cc-967f-fe7d19d6e10a)
>
>El profesor, ha dicho que puede ser este:
>
>![image](https://github.com/user-attachments/assets/bb49a054-658c-48dd-9e15-7feec89be353)
>

> [!IMPORTANT]
> New accounts only support launch templates
Starting on October 1, 2024, Amazon EC2 Auto Scaling will no longer support the creation of launch configurations for new accounts. Existing environments will not be impacted. For more information about other situations that are impacted, including temporary option settings required for new accounts, refer to Launch templates  in the Elastic Beanstalk Developer Guide.
>
> https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-cfg-autoscaling-launch-templates.html
> 
## Hacer Update al grupo de Seguridad y ELB.
