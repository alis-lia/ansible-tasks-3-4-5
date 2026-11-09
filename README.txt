=====================================================================
 ASO - Grupo 1 y 2 - Tareas 3, 4 y 5
 Grupo de Entrega - 8
 Despliegue remoto LAMP + phpMyAdmin + Samba + FTP + PostgreSQL
 con Ansible (roles) sobre openSUSE Leap 16.0
=====================================================================

a) REPOSITORIO GITHUB
---------------------
https://github.com/alis-lia/ansible-tasks-3-4-5
(Compartido con gcussi@yahoo.com)

b) PASOS PARA EJECUTAR EL PROYECTO
----------------------------------
Requisitos:
  - CONTROL: openSUSE Leap 16.0 con ansible, sshpass y git
        zypper in -y ansible sshpass git
    Solo si al ejecutar aparece un error de modulo no encontrado
    (por ejemplo, en Ubuntu/WSL con ansible-core), instalar las colecciones:
        ansible-galaxy collection install community.general community.mysql community.postgresql ansible.posix
  - DESTINO: openSUSE Leap 16.0 LIMPIO (sin aplicaciones), con SSH para
    root habilitado y acceso a internet.
    Si no esta habilitado, ejecutar en el DESTINO:
        echo "PermitRootLogin yes" > /etc/ssh/sshd_config.d/root.conf
        systemctl enable --now sshd && systemctl restart sshd

1. Clonar el repositorio:
     git clone https://github.com/alis-lia/ansible-tasks-3-4-5.git
     cd ansible-tasks-3-4-5

2. Configurar el archivo hosts: poner la IP del DESTINO en
     inventory/hosts.yml   (campo ansible_host)

3. Ejecutar:
     ansible-playbook site.yml -k --ask-vault-pass

   Pedira dos contrasenas:
     SSH password   -> contrasena de root del DESTINO
     Vault password -> se entrega junto al README en Classroom

   Opcional, ejecutar solo una parte (tags: lamp, apache, mariadb,
   phpmyadmin, samba, ftp, postgresql):
     ansible-playbook site.yml -k --ask-vault-pass --tags samba

Verificacion (desde otra maquina de la red):
   - Apache/PHP:   http://IP_DESTINO/
   - phpMyAdmin:   http://IP_DESTINO/phpmyadmin   (usuario aso_user)
   - Samba:        smbclient //IP_DESTINO/compartido -U alumno
   - FTP:          ftp IP_DESTINO                 (usuario alumno)
   - PostgreSQL:   psql "host=IP_DESTINO dbname=aso_pgdb user=aso_pg sslmode=require"

Las contrasenas de los servicios estan en el vault:
  ansible-vault view group_vars/all/vault.yml

Nota: si Ansible avisa que ignora ansible.cfg por estar en un directorio
con permisos de escritura para todos, ejecutar: chmod 755 .

ESTRUCTURA
----------
ansible.cfg              Configuracion (pipelining, forks, tiempos por tarea)
inventory/hosts.yml      Servidores (sin usuario ni contrasena)
group_vars/all/          Variables y vault cifrado
site.yml                 Playbook principal
roles/
  apache_php/            Tarea 3: Apache + PHP
  mariadb/               Tarea 3: configuracion de MariaDB
  phpmyadmin/            Tarea 3: phpMyAdmin
  samba/                 Tarea 3: usuario de sistema, usuario Samba,
                                  directorio y smb.conf (create mask)
  vsftpd/                Tarea 3: FTP (plantilla vsftpd.conf)
  postgresql/            Tarea 5: PostgreSQL, php8-pgsql, usuario, base,
                                  conexion remota cifrada (SSL)

Tarea 4: repositorio en GitHub; plantillas en roles/*/templates
           (smb.conf.j2, vsftpd.conf.j2, config.inc.php.j2, ...)

MEJORAS APLICADAS
-----------------
- Inventario sin usuario/contrasena visible: solo IPs; la contrasena SSH
  se pide al ejecutar (-k) y los secretos estan cifrados con Ansible Vault.
- Tiempo de respuesta: pipelining, ControlPersist, forks=10,
  gather_facts desactivado, paquetes instalados en una sola transaccion,
  y sin "zypper update" completo. El callback profile_tasks muestra
  cuanto tarda cada tarea.
- Roles independientes y reutilizables, con plantillas Jinja2.



=====================================================================
 DESTINO
=====================================================================

su -
echo "PermitRootLogin yes" > /etc/ssh/sshd_config.d/root.conf
systemctl enable --now sshd && systemctl restart sshd
systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target packagekit
systemctl restart NetworkManager
ip -4 a
ping -c 2 google.com
rpm -q apache2 mariadb samba vsftpd postgresql-server


=====================================================================
 MASTER
=====================================================================

su -c "zypper in -y ansible sshpass git samba-client lftp postgresql"
git clone https://github.com/alis-lia/ansible-tasks-3-4-5.git
cd ansible-tasks-3-4-5
chmod 755 .
cat README.txt
nano inventory/hosts.yml
ansible all -k --ask-vault-pass -m ping
time ansible-playbook site.yml -k --ask-vault-pass
time ansible-playbook site.yml -k --ask-vault-pass

SSH password   -> contrasena de root del DESTINO
Vault password -> se entrega junto al README en Classroom


=====================================================================
 VERIFICACION (MASTER)
=====================================================================

ansible all -k --ask-vault-pass -m shell -a "systemctl is-active apache2 mariadb smb vsftpd postgresql"
ansible-vault view group_vars/all/vault.yml
smbclient //IP_DESTINO/compartido -U alumno -c "put /etc/hostname prueba.txt; ls"
ssh root@IP_DESTINO "ls -l /srv/samba/compartido"
lftp -u alumno -e "ls; bye" IP_DESTINO
psql "host=IP_DESTINO dbname=aso_pgdb user=aso_pg sslmode=require" -c "\conninfo"
psql "host=IP_DESTINO dbname=aso_pgdb user=aso_pg sslmode=disable" -c "\conninfo"

Navegador:
http://172.20.10.6/
http://172.20.10.6/phpmyadmin

