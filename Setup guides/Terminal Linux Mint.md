# Configuración de mi Terminal - Linux Mint 22.x

> Entorno de terminal moderno basado en **Zsh + Oh My Zsh + Oh My Posh**, pensado para desarrollo y administración de sistemas.

---

# Software utilizado

| Herramienta        | Descripción                                                    |
| ------------------ | -------------------------------------------------------------- |
| Zsh                | Shell moderna con autocompletado y mejor experiencia que Bash. |
| Oh My Zsh          | Framework para administrar la configuración y plugins de Zsh.  |
| Oh My Posh         | Prompt altamente personalizable.                               |
| FiraCode Nerd Font | Fuente con iconos para el prompt y herramientas modernas.      |
| eza                | Reemplazo moderno de `ls`.                                     |
| bat                | Reemplazo de `cat` con resaltado de sintaxis.                  |
| zoxide             | Navegación inteligente entre directorios.                      |
| fzf                | Buscador difuso para archivos, historial y autocompletado.     |
| btop               | Monitor interactivo del sistema.                               |
| neofetch           | Información del sistema en la terminal.                        |

---

# 1. Actualizar el sistema

```bash
sudo apt update && sudo apt upgrade -y
```

---

# 2. Instalar dependencias

```bash
sudo apt install -y \
git \
curl \
wget \
unzip \
build-essential
```

---

# 3. Instalar Zsh

```bash
sudo apt install -y zsh
```

Verificar:

```bash
zsh --version
which zsh
```

---

# 4. Configurar Zsh como shell por defecto

```bash
chsh -s $(which zsh)
```

Cerrar sesión y volver a iniciar.

Comprobar:

```bash
echo $0
echo $SHELL
getent passwd $USER
```

---

# 5. Instalar Oh My Zsh

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

---

# 6. Instalar Oh My Posh

```bash
curl -s https://ohmyposh.dev/install.sh | bash -s
```

Verificar:

```bash
oh-my-posh version
```

---

# 7. Descargar los temas

```bash
mkdir -p ~/.poshthemes

curl -L \
https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/themes.zip \
-o ~/.poshthemes/themes.zip

unzip ~/.poshthemes/themes.zip -d ~/.poshthemes

chmod u+rw ~/.poshthemes/*.omp.json

rm ~/.poshthemes/themes.zip
```

---

# 8. Instalar la Nerd Font

```bash
oh-my-posh font install
```

Seleccionar:

```
FiraCode Nerd Font
```

Después configurar la terminal para utilizar esa fuente.

---

# 9. Instalar plugins

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions \
${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting \
${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

---

# 10. Instalar herramientas adicionales

```bash
sudo apt install -y \
eza \
bat \
fzf \
btop \
neofetch
```

Instalar zoxide:

```bash
curl -sSfL https://raw.githubusercontent.com/ajeetdsouza/zoxide/main/install.sh | sh
```

---

# 11. Configurar ~/.zshrc

Plugins:

```zsh
plugins=(
    git
    sudo
    extract
    colored-man-pages
    command-not-found
    zsh-autosuggestions
    zsh-syntax-highlighting
)
```

Inicializar zoxide:

```zsh
eval "$(zoxide init zsh)"
```

Inicializar Oh My Posh:

```zsh
eval "$(oh-my-posh init zsh --config ~/.poshthemes/kali.omp.json)"
```

Alias recomendados:

```zsh
alias ls='eza --icons'
alias ll='eza -lah --icons --git'
alias la='eza -a --icons'
alias tree='eza --tree --icons'

alias cat='batcat'

alias cls='clear'

alias sysinfo='neofetch'

alias rm='rm -I'
alias cp='cp -i'
alias mv='mv -i'

alias myip='curl ifconfig.me'
```

---

# 12. Aplicar la configuración

```bash
source ~/.zshrc
```

---

# Comandos útiles

| Comando          | Descripción                                 |
| ---------------- | ------------------------------------------- |
| `ll`             | Lista archivos con iconos.                  |
| `tree`           | Árbol de directorios.                       |
| `btop`           | Monitor del sistema.                        |
| `neofetch`       | Información del sistema.                    |
| `z proyecto`     | Ir rápidamente a un directorio frecuente.   |
| `batcat archivo` | Mostrar archivos con resaltado de sintaxis. |

---

# Resultado

* ✅ Zsh
* ✅ Oh My Zsh
* ✅ Oh My Posh
* ✅ Tema Kali
* ✅ FiraCode Nerd Font
* ✅ eza
* ✅ bat
* ✅ zoxide
* ✅ fzf
* ✅ btop
* ✅ neofetch
* ✅ Plugins personalizados
* ✅ Alias personalizados
