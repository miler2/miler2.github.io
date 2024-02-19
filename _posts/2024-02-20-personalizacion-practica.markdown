---
layout: post
title:  "Practica docker"
date:   2024-02-19 06:46:41 -0600
categories: jekyll update
---

# Terraform
Para las practicas de terraform y los archivos .yml, he usado la extensión <u>"HashiCorp Terraform"</u>

## Instalación de terraform
### Para la instalación en **Linux**
En esta [página web](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli) tenemos una guía con los comandos que hacen falta para instalar terraform.

### Para la instalación en **windows**
Primero tenemos que instalarnos el paquete en esta [url](https://developer.hashicorp.com/terraform/install) e instalarnos la versión para windows.

Luego tendremos que irnos a este directorio y mover nuestro archivo .exe de terraform aquí.

```
%USERPROFILE%\AppData\Local\Microsoft\WindowsApps
```

# Teoría:
## Variables
Las variables se definen en un archivo llamado **variables.tf**. No hace falta decirle qué archivo es, ya que busca un archivo con este nombre él mismo.

Una variable se define así:

```
variable "region" {  
  description = "Región de AWS donde se creará la instancia"  
  type        = string  
  default     = "us-east-1"
}
```

- La palabra "region" declara el nombre de la variable.
- **default** es el valor que guarda la variable.

Para llamar la variable tendremos que usar esta sintaxis:

> var.region

## Arrays
Para guardar un array se hace así:

```
variable "allowed_ingress_ports" {
  description = "Puertos de entrada del grupo de seguridad"
  type        = list(number)
  default     = [22, 80, 443]
}
```

Aquí tenemos declarados los valores de 22, 80 y 443. Si queremos llamar a uno de éstos tendremos que usar:

> var.allowed_ingress_ports[1]

El número indica qué posición tiene la variable dentro del array y nos saca ese valor concreto.

### *Bucles*
Podríamos hacer un bucle utilizando un array de esta manera:

```
resource "aws_security_group_rule" "ingress" {
  security_group_id = aws_security_group.security_group_balancer.id
  type              = "ingress"

  count       = length(var.allowed_ingress_ports)
  from_port   = var.allowed_ingress_ports[count.index]
  to_port     = var.allowed_ingress_ports[count.index]
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]
}
```
- "count       = length(var.allowed_ingress_ports)"

Me realiza el bucle tantas veces como variables hay en el array definido por mi.

- "from_port   = var.allowed_ingress_ports[count.index]"

Me está buscando el valor del array y [count.index] me selecciona el primer valor si es el primer intento, el segundo si es el segundo intento...


# Acciones en un archivo terraform
El comando más importante para un archivo terraform es "provider" porque nos dirá qué proveedor estamos usando, y la forma en que interpretará lo escrito después varia.

Ejemplo:

```
provider "aws" {
  region = us-east-1
}
```

## Crear un grupo de seguridad
Si queremos crear un grupo de seguridad tendremos que usar un comando con la siguiente sintaxis.

```
resource "aws_security_group" "sg_ejemplo_04" {
  name        = "sg_ejemplo_04"
  description = "Grupo de seguridad para la instancia de ejemplo 04"

  # Reglas de entrada para permitir el tráfico SSH, HTTP y HTTPS
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Reglas de salida para permitir todas las conexiones salientes
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

Aquí estamos creando un grupo de seguridad llamado "sg_ejemplo_04" y le estamos dando los puertos que queremos que tenga abiertos tanto en salida como en entrada.

- Entrada: **ingress**
- Salida: **egress**

Si queremos habilitar **todos los puertos** tendremos que usar el valor de "0" en vez del número de puerto, además de usar un valor de protocolo de "-1" (un ejemplo aparece más arriba en los valores de egress).

Si queremos proporcionarle los valores usando variables tendremos que usar un array, como ya hemos hablado previamente en este documento.

Se haría de la siguiente manera:

```
# Variable guardada en un archivo "variables.tf"

variable "allowed_ingress_ports" {
  description = "Puertos de entrada del grupo de seguridad"
  type        = list(number)
  default     = [22, 80, 443]
}


# Creación del grupo de seguridad

resource "aws_security_group_rule" "ingress" {
  security_group_id = aws_security_group.security_group_balancer.id
  type              = "ingress"

  count       = length(var.allowed_ingress_ports)
  from_port   = var.allowed_ingress_ports[count.index]
  to_port     = var.allowed_ingress_ports[count.index]
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]
}
```
Este ejemplo es únicamente para declarar los valores de entrada, no los de salida, por lo que han de hacerse dos bucles y dos arrays, uno para los de entrada, y otro para los de salida.

La explicación de este ejemplo exacto se ha hecho más arriba en la explicación de arrays.

## Creación de instancias
Este es un ejemplo de cómo crear una instancia:

```
resource "aws_instance" "instancia_ejemplo_04" {
  ami             = "ami-00874d747dde814fa"
  instance_type   = "t2.small"
  key_name        = "vockey"
  security_groups = [aws_security_group.sg_ejemplo_04.name]

  user_data = file("scripts/install_docker.sh")

  tags = {
    Name = "instancia_ejemplo_04"
  }
}
```

El nombre que le hemos puesto a esta máquina es "instancia_ejemplo_04".

- ami             = "ami-00874d747dde814fa"

    Hace referencia a qué sistema operativo queremos elegir (ubuntu, redhat...).

- instance_type   = "t2.small"

    Indica el tipo de instancia que queremos (este caso es el que tiene 2gb de ram).

- key_name        = "vockey"

    Indica el nombre de nuestra llave de seguridad de aws. Aquí está usando la de por defecto.

- security_groups = [aws_security_group.sg_ejemplo_04.name]

    Aquí me está guardando el nombre del grupo de seguridad.
    
    La forma en que se hace aquí es buscando en el grupo de seguridad **(que ya está creado)** y luego busca su nombre, y guarda eso aquí como su valor.

## Asignar IPs elásticas
Este es un ejemplo de cómo asignar una ip elástica a una máquina ya creada.

```
resource "aws_eip" "ip_elastica" {
  instance = aws_instance.instancia_ejemplo_06.id
}
```

# Comandos básicos

### Inicializar el directorio de trabajo

En primer lugar hay que descargar los plugins necesarios del proveedor que se especifica en el archivo de configuración.

```
terraform init
```

### Formatear y validar el archivo de configuración

Terraform nos proporciona un comando para formatear el archivo de configuración para que sea más legible. Algunas de las tareas que realiza este comando son, ajustar la indentación, ordenar los argumentos de los bloques de configuración, etc.

```
terraform fmt
```

Para validar la sintaxis del archivo de configuración podemos utilizar el siguiente comando.

```
terraform validate
```

### Mostrar los cambios que se van a realizar

Compara la configuración del archivo de Terraform con la que existe actualmente en el proveedor de infraestructura y muestra las acciones que se tienen que realizar para conseguir la configuración deseada. Permite al usuario verificar los cambios antes de aplicarlos en el proveedor.

```
terraform plan
```

### Aplicar los cambios

Crea los recursos del archivo de configuración en su cuenta de AWS.

```
terraform apply
```

Si queremos crear los recursos sin tener que escribir `yes` para confirmar la ejecución del comando, podemos utilizar la opción `-auto-approve`.

```
terraform apply -auto-approve
```

### Mostrar el estado actual de los recursos

Muestra los recursos creados en el proveedor y su estado actual.

``` 
terraform show
```

### Eliminar los recursos

Elimina los recursos indicados en el proveedor.

``` 
terraform destroy
```

## Notas:
Para usar terraform tenemos que estar dentro de una ruta en la que hay un archivo .tf, y luego podremos usar el comando terraform init.