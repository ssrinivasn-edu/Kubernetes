
## Git Installation

### Windows
1. Visit the official Git website at https://git-scm.com/download/win.
2. Download the "64-bit Git for Windows Setup" installer.
3. Run the installer.
4. Follow the prompts in the installer, using the default settings.
5. On the "Select Components" screen, make sure "Git Bash Here," "Git GUI Here," and "Associate .git* configuration files with the default text editor" options are selected.
6. Continue with the installation, accepting the license and choosing an appropriate installation location if desired.
7. On the "Adjusting your PATH environment" screen, select the recommended option, which is "Use Git from the Windows Command Prompt."
8. Complete the installation process.

### macOS
1. Visit the official Git website at https://git-scm.com/download/mac.
2. Download the macOS installer.
3. Open the installer.
4. Follow the prompts in the installer, using the default settings.
5. Review and accept the software license agreement.
6. Choose the destination where you want to install Git, or use the default location.
7. Optionally, select the components you want to install. The default selection should be sufficient for most users.
8. Continue with the installation.
9. Open Terminal and verify that Git is installed by running the command: `git --version`.

### Linux
- **Ubuntu/Debian:**
  ```
  sudo apt update
  sudo apt install git
  ```

- **Fedora:**
  ```
  sudo dnf install git
  ```

- **CentOS/RHEL:**
  ```
  sudo yum install git
  ```

- **Arch Linux:**
  ```
  sudo pacman -Syu git
  ```

After the installation is complete, you can verify that Git is installed by running the command: `git --version`.
