# BlueZ Ansible Role for Alpine Linux

This repository contains an Ansible role to install and configure BlueZ, the Bluetooth protocol stack, on Alpine Linux. The role ensures idempotency and is tested using Molecule with a Dockerized Alpine Linux environment. This `README.md` provides instructions for running the playbook on `localhost` and testing it with Molecule.

## Prerequisites

Before running the playbook or tests, ensure the following are installed on your local machine:

- **Ansible**: Version 2.10 or higher.
- **Python**: Version 3.11 or higher (required for Ansible and Molecule).
- **Docker**: Required for Molecule tests.
- **Molecule**: For running automated tests.
- **ansible-galaxy**: To install collection dependencies.
- **Alpine Linux**: The target `localhost` must be running Alpine Linux (tested with version 3.18).

### Installation of Prerequisites

1. **Install Python**:
   ```bash
   sudo apk add python3 py3-pip
   ```

2. **Install Ansible**:
   ```bash
   pip3 install ansible
   ```

3. **Install Molecule and Docker**:
   ```bash
   pip3 install molecule molecule-docker docker
   ```

4. **Install Ansible Collections**:
   ```bash
   ansible-galaxy collection install -r molecule/default/requirements.yml
   ```

5. **Install Docker on Alpine Linux**:
   ```bash
   sudo apk add docker
   sudo service docker start
   sudo rc-update add docker
   ```

6. **Ensure SSH Access for Localhost**:
   - Ensure SSH is enabled on your Alpine Linux machine:
     ```bash
     sudo apk add openssh
     sudo service sshd start
     sudo rc-update add sshd
     ```
   - Verify you can SSH to `localhost`:
     ```bash
     ssh localhost
     ```
   - If you encounter permission issues, ensure your user has SSH access or use `sudo` for Ansible commands.

## Directory Structure

The repository is organized as follows:

```
project/
├── .github/
│   └── workflows/
│       └── molecule.yml       # GitHub Action for CI
├── molecule/
│   └── default/
│       ├── molecule.yml       # Molecule configuration
│       ├── converge.yml       # Molecule playbook to apply the role
│       ├── verify.yml         # Molecule verification tests
│       └── requirements.yml   # Ansible collection dependencies
├── roles/
│   └── bluez/
│       ├── defaults/
│       │   └── main.yml       # Default variables
│       ├── handlers/
│       │   └── main.yml       # Service handlers
│       └── tasks/
│           └── main.yml       # Main task list
├── site.yml                  # Entry point playbook
├── README.md                 # This file
```

## Running the Playbook on Localhost

To apply the BlueZ role to your local Alpine Linux machine, follow these steps:

1. **Clone the Repository**:
   ```bash
   git clone <repository-url>
   cd <repository-directory>
   ```

2. **Create an Inventory File**:
   Create a file named `inventory.yml` with the following content:
   ```yaml
   all:
     hosts:
       localhost:
         ansible_connection: local
         ansible_python_interpreter: /usr/bin/python3
   ```

   This configures Ansible to run on `localhost` without SSH, using the local Python interpreter.

3. **Run the Playbook**:
   Execute the `site.yml` playbook to apply the BlueZ role:
   ```bash
   ansible-playbook -i inventory.yml site.yml
   ```

   - The playbook installs `bluez` and `rfkill`, adds the user to the `lp` group, loads the `btusb` kernel module, configures it for boot, unblocks Bluetooth if soft-blocked, and ensures the Bluetooth service is running and enabled.
   - The playbook is idempotent, so running it multiple times will not cause unintended changes.

4. **Verify the Installation**:
   After running the playbook, check the Bluetooth service and controller:
   ```bash
   sudo service bluetooth status
   bluetoothctl list
   bluetoothctl show
   ```
   - Ensure the service is running and the controller is powered on.
   - Note: If no Bluetooth hardware is present, `bluetoothctl list` may return empty, which is expected.

## Running Molecule Tests Locally

To test the BlueZ role using Molecule, follow these steps:

1. **Ensure Docker is Running**:
   ```bash
   sudo service docker status
   sudo service docker start
   ```

2. **Navigate to the Repository**:
   ```bash
   cd <repository-directory>
   ```

3. **Run Molecule Tests**:
   ```bash
   molecule test
   ```

   - This command runs the full Molecule test suite:
     - **Create**: Sets up a Docker container with Alpine Linux 3.18.
     - **Converge**: Applies the BlueZ role.
     - **Idempotence**: Verifies the role is idempotent (no changes on second run).
     - **Verify**: Runs tests to check service status, module loading, user group membership, and Bluetooth controller state.
     - **Destroy**: Cleans up the Docker container.
   - Note: The Bluetooth controller test may fail in Docker if no hardware is passed through, as Docker containers typically lack Bluetooth devices.

4. **Debugging Test Failures**:
   - View logs in the terminal output.
   - Run `molecule converge` to apply the role without destroying the container, then use `molecule login` to inspect the container manually.

## Notes

- **Idempotency**: The role is designed to be idempotent, checking existing states (packages, groups, modules, services) before applying changes.
- **Bluetooth Hardware**: Some verification steps (e.g., `bluetoothctl list`) require Bluetooth hardware. In virtualized environments like Docker, these may fail, which is expected.
- **Privileges**: Running on `localhost` requires `sudo` privileges for tasks like package installation and service management. Ensure your user has sufficient permissions or use `ansible-playbook -K` to provide a sudo password.
- **GitHub Actions**: The `.github/workflows/molecule.yml` file automates testing on push/pull requests to the `main` branch. Local tests replicate this CI process.

## Troubleshooting

- **SSH Issues**: If SSH to `localhost` fails, ensure `sshd` is running and your user is authorized. Alternatively, use `ansible_connection: local` as shown in the inventory.
- **Docker Issues**: Ensure Docker is installed and the service is running. Check `docker info` for status.
- **Molecule Failures**: Check `molecule/default/verify.yml` for specific test failures. Run `molecule --debug test` for verbose output.
- **Bluetooth Service**: If the service fails to start, verify Bluetooth hardware is present and not hard-blocked (`rfkill list bluetooth`).

For further assistance, refer to the Ansible and Molecule documentation or open an issue in the repository.