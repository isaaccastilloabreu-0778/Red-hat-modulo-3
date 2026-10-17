# ==========================================
# PRÁCTICA 1: CONFIGURACIÓN DE GRUB Y CONTRASEÑA
# ==========================================

# 1. Configurar el archivo de GRUB
sudo nano /etc/default/grub
# (Cambiar GRUB_TIMEOUT=20 y guardar)

# 2. Actualizar GRUB en Red Hat
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# 3. Recuperar contraseña de root (después de reiniciar y usar rd.break en GRUB)
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
touch /.autorelabel
exit
exit


# ==========================================
# PRÁCTICA 2: SCRIPTS DE BACKUP Y RED
# ==========================================

# --- Script 1: Backup comprimido (backup.sh) ---
nano backup.sh
# (Pega el contenido y guarda)
#!/bin/bash
FECHA=$(date +"%d-%m-%Y_%H-%M")
DESTINO="/tmp/backup_${USER}_${FECHA}.tar.gz"
echo "Creando respaldo de la carpeta /home/$USER..."
tar -czf "$DESTINO" /home/$USER 2>/dev/null
echo "¡Backup completado con éxito!"
echo "Archivo guardado en: $DESTINO"

# Dar permisos y ejecutar script 1
chmod +x backup.sh
./backup.sh


# --- Script 2: Copiar red al Escritorio (red.sh) ---
nano red.sh
# (Pega el contenido y guarda)
#!/bin/bash
if [ -d "/home/$USER/Escritorio" ]; then
    DIRECTORIO_ESCRITORIO="/home/$USER/Escritorio"
else
    DIRECTORIO_ESCRITORIO="/home/$USER/Desktop"
fi
read -p "Introduce el nombre que deseas para el archivo: " NOMBRE_ARCH
if [ -z "$NOMBRE_ARCH" ]; then
    echo "Error: No ingresaste ningún nombre."
    exit 1
fi
RUTA_FINAL="$DIRECTORIO_ESCRITORIO/$NOMBRE_ARCH.txt"
ifconfig > "$RUTA_FINAL" 2>/dev/null || ip a > "$RUTA_FINAL"
echo "¡Archivo creado exitosamente!"
echo "Ubicación: $RUTA_FINAL"

# Dar permisos y ejecutar script 2
chmod +x red.sh
./red.sh


# ==========================================
# PRÁCTICA 3: CONEXIÓN SSH Y LLAVES (HOST WINDOWS Y VM)
# ==========================================

# --- En la Máquina Virtual (Red Hat) ---
# 1. Obtener la IP
ip a

# 2. Habilitar e iniciar el servicio SSH
sudo systemctl enable --now sshd


# --- En la Máquina Host (Windows - PowerShell o CMD) ---
# 1. Probar conectividad con la VM
ping <IP_DE_TU_VM>

# 2. Generar el par de llaves (dejar la frase de contraseña en blanco)
ssh-keygen -t rsa -b 4096

# 3. Copiar la llave pública al servidor (desde Windows)
scp ~/.ssh/id_rsa.pub tu_usuario@<IP_DE_TU_VM>:/tmp/authorized_keys


# --- De vuelta en la Máquina Virtual (Red Hat) ---
# Configurar los permisos de seguridad de las llaves
mkdir -p ~/.ssh
cat /tmp/authorized_keys >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
rm -f /tmp/authorized_keys


# --- Prueba final (Desde Windows) ---
# Conectarse sin contraseña de forma automática
ssh tu_usuario@<IP_DE_TU_VM>
