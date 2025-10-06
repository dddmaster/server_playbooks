# server_playbooks
Collection of server playbooks | ai generated 

# Usage Instructions:

## Running the Playbook:

Install Ansible and the community.general collection: ansible-galaxy collection install community.general.
Run: ansible-playbook site.yml -i <inventory>.


## Running Molecule Tests:

Install Molecule and Docker: pip install molecule molecule-docker ansible.
Place files in the structure above.
Run: molecule test to verify idempotency and functionality.


## Notes:

The playbook remains idempotent; multiple runs will not cause unintended changes.
Handlers ensure service actions are consolidated and only executed when needed.
Molecule tests validate idempotency by running the role twice and checking for no changes on the second run.
The verify.yml playbook ensures the desired state (service running, module loaded, user in group, etc.).
Bluetooth controller detection may fail in Docker if no hardware is present; adjust expectations for virtualized environments.