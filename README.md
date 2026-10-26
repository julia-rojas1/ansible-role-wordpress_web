# wordpress_web

Rol de Ansible que instala y configura el frontend web de WordPress: Apache, PHP, descarga de WordPress, `wp-config.php` y VirtualHost. La base de datos **no** se instala con este rol; debe existir y ser accesible.

## Requisitos

- Ansible >= 2.9
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

1. Carga variables según el SO.
2. Instala Apache, PHP y librerías necesarias (actualiza caché de apt en Debian/Ubuntu).
3. Habilita `mod_rewrite` (Debian/Ubuntu) e inicia/habilita el servicio web.
4. Crea el directorio y descarga/descomprime WordPress.
5. Genera `wp-config.php` y el VirtualHost desde templates.
6. Debian: habilita el sitio y deshabilita el VirtualHost por defecto.
7. RedHat: abre HTTP en firewalld y activa el booleano SELinux `httpd_can_network_connect_db`.

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

Archivos: `molecule/default/{molecule,converge,verify}.yml`.

```bash
pip install molecule molecule-plugins[docker] ansible-lint
molecule test          # ciclo completo
molecule converge      # solo aplicar el rol
molecule verify        # solo verificaciones
molecule destroy       # limpiar contenedores
```

## Licencia

MIT

## Autor

Julia Rojas
