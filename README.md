# wordpress_web

Rol de Ansible que instala y configura el frontend web de WordPress: Apache, PHP, descarga de WordPress, `wp-config.php` y VirtualHost. La base de datos **no** se instala con este rol; debe existir y ser accesible.

## Requisitos

- `ansible-core` >= 2.15
- Sistemas soportados: Ubuntu/Debian y EL (Rocky Linux 9 / RedHat / CentOS).
- Una base de datos MySQL/MariaDB accesible con las credenciales indicadas.
- Para los tests: Docker, Molecule y `ansible-lint`.

## Variables del rol

Definidas en `defaults/main.yml`:

| Variable | Valor por defecto | Descripción |
|---|---|---|
| `wp_version` | `latest` | Versión de WordPress a descargar |
| `wp_install_dir` | `/var/www/wordpress` | Directorio de instalación |
| `wp_port` | `80` | Puerto del VirtualHost |
| `wp_server_name` | `_` | `ServerName` del VirtualHost |
| `wp_db_name` | `wordpress_db` | Nombre de la base de datos |
| `wp_db_user` | `wp_user` | Usuario de la base de datos |
| `wp_db_password` | `Password123!` | Contraseña de la base de datos (**cambiar en producción**, idealmente con Ansible Vault) |
| `wp_db_host` | `localhost` | Host de la base de datos |

Variables específicas por familia de SO en `vars/Debian.yml` y `vars/RedHat.yml` (paquetes PHP, nombre del servicio web, etc.).

## Qué hace el rol

1. Carga dinámicamente las variables correspondientes a la familia del SO (`Debian` o `RedHat`).
2. Actualiza la caché de paquetes (`apt`) en Debian/Ubuntu e instala Apache, PHP y sus módulos requeridos.
3. Habilita el módulo `rewrite` de Apache en Debian/Ubuntu.
4. Asegura que el servicio web (`apache2` o `httpd`) esté iniciado y habilitado en el arranque.
5. Crea el directorio de destino y descarga/descomprime la versión solicitada de WordPress.
6. Genera el archivo `wp-config.php` (con permisos `0640`) y el VirtualHost desde plantillas Jinja2.
7. En Debian/Ubuntu: activa el nuevo VirtualHost y desactiva el sitio por defecto (`000-default.conf`).
8. En RedHat/Rocky (en entornos no-Docker): habilita el puerto HTTP en `firewalld` y el booleano SELinux `httpd_can_network_connect_db`.

## Dependencias

Ninguna.

## Ejemplo de playbook

```yaml
- hosts: wordpress_hosts
  become: true
  roles:
    - role: wordpress_web
      vars:
        wp_version: "6.5.5"
        wp_db_host: "10.0.0.10"
        wp_db_password: "{{ vault_wp_db_password }}"
        wp_server_name: "blog.example.com"
```

## Tests con Molecule

El escenario `default` usa el driver Docker con dos plataformas con systemd:

- `ubuntu-2204` (`geerlingguy/docker-ubuntu2204-ansible`)
- `rocky9` (`geerlingguy/docker-rockylinux9-ansible`)

Estructura de pruebas en molecule/default/:
- molecule.yml: Configuración de plataformas y driver.  
- converge.yml: Ejecución del rol sobre los contenedores.  
- verify.yml: Aserciones de servicios activos, permisos de archivos (0640) y respuesta HTTP 200.  


### Comandos de prueba
Si utilizas *Poetry*:
```bash
poetry run molecule test        # Ciclo completo de testing (convergencia, idempotencia y verificación)
poetry run molecule converge    # Solo aplica el rol (útil para desarrollo)
poetry run molecule verify      # Ejecuta las aserciones de verificación
poetry run molecule destroy     # Limpia los contenedores de prueba
```
Si utilizas *pip*
```bash
pip install molecule "molecule-plugins[docker]" ansible-lint
molecule test
```

## Calidad del código
El rol pasa todos los controles de calidad exigidos por ansible-lint con perfil de producción:
```bash
poetry run ansible-lint
```

## Licencia

MIT

## Autor

Julia Rojas
