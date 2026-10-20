# Ansible Website Deployment Practical

## Aim
To deploy a simple HTML website using Ansible automation on Ubuntu running through WSL 2 on Windows.

## Requirements
- Windows PC
- WSL 2
- Ubuntu Linux distribution
- Visual Studio Code (optional)
- Ansible
- Apache HTTP Server

> **Important:** Run Windows/WSL installation commands in PowerShell. Run Linux and Ansible commands in the Ubuntu terminal.

---

## Part 1: Install Ubuntu (WSL 2)

1. Open **PowerShell as Administrator**.
2. Install Ubuntu:

   ```powershell
   wsl --install -d Ubuntu
   ```

3. Restart the PC if Windows asks you to.
4. Open Ubuntu from the Start menu if it does not launch automatically.
5. Create a Linux username and password when prompted. Password characters will not appear while typing; this is normal.
6. Verify the Linux username:

   ```bash
   whoami
   ```

If WSL reports that Ubuntu is already installed, check distributions with:

```powershell
wsl --list --verbose
```

## Part 2: Install Ansible

Run these commands in the **Ubuntu terminal**:

```bash
sudo apt update
sudo apt install ansible -y
ansible --version
```

The last command should display Ansible version information.

## Part 3: Create the Project

Create and open the project directory:

```bash
mkdir -p ~/ansible
cd ~/ansible
```

If Visual Studio Code and its WSL extension are installed, open the folder with:

```bash
code .
```

If `code .` is unavailable, create the files using `nano` as described below.

### 1. Create `inventory`

```bash
nano inventory
```

Add:

```ini
[webserver]
localhost ansible_connection=local
```

Save with **Ctrl+O**, press **Enter**, then exit with **Ctrl+X**.

### 2. Create `index.html`

```bash
nano index.html
```

Add:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Website</title>
</head>
<body>
    <h1>Ansible</h1>
    <p>My website has been deployed successfully!</p>
</body>
</html>
```

Save with **Ctrl+O**, press **Enter**, then exit with **Ctrl+X**.

### 3. Create `playbook.yml`

```bash
nano playbook.yml
```

Add:

```yaml
---
- name: Deploy Website
  hosts: webserver
  connection: local
  become: true

  tasks:
    - name: Create website directory
      file:
        path: /var/www/html
        state: directory

    - name: Copy Website
      copy:
        src: index.html
        dest: /var/www/html/index.html

    - name: Show success message
      debug:
        msg: "Website deployed successfully!"
```

Save with **Ctrl+O**, press **Enter**, then exit with **Ctrl+X**.

The `become: true` setting lets Ansible run tasks with elevated privileges. The `file` task ensures that the destination directory exists.

## Part 4: Test and Run Ansible

Make sure the terminal is in the project directory (`~/ansible`).

### 1. Check the inventory

```bash
ansible-inventory -i inventory --list
```

### 2. Test local connectivity

```bash
ansible webserver -i inventory -m ping
```

Expected result includes:

```text
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### 3. Run the playbook

```bash
ansible-playbook -i inventory playbook.yml --ask-become-pass
```

Enter your Ubuntu password when prompted for the become password.

A successful run should show the tasks completing without failures and a message similar to:

```text
"msg": "Website deployed successfully!"
```

The exact number of changed tasks may vary on subsequent runs because Ansible is designed to be idempotent.

## Part 5: Install Apache and View the Website

The playbook copies the HTML file, but it does not start a web server. Install Apache to serve the page.

### 1. Install Apache

```bash
sudo apt install apache2 -y
```

### 2. Start Apache

```bash
sudo service apache2 start
```

### 3. Check Apache status if needed

```bash
sudo service apache2 status
```

### 4. Open the website

Open a browser on Windows and visit:

http://localhost

You should see the heading **Ansible** and the message **My website has been deployed successfully!**

If the page does not appear, confirm Apache is running and check whether another service is already using port 80.

## Project Structure

```text
ansible/
├── inventory
├── index.html
└── playbook.yml
```

## Important Commands for the Exam

| Command | Purpose |
|---|---|
| `ansible --version` | Checks Ansible installation |
| `ansible-inventory -i inventory --list` | Displays inventory details |
| `ansible webserver -i inventory -m ping` | Tests connectivity to the managed host |
| `ansible-playbook -i inventory playbook.yml --ask-become-pass` | Runs the deployment playbook |
| `sudo service apache2 start` | Starts Apache |
| `sudo service apache2 status` | Checks Apache status |

## Viva / Exam Write-up

**Aim:** To deploy a simple HTML website using Ansible automation.

**Procedure:**
1. Install Ubuntu using WSL 2.
2. Install Ansible in Ubuntu.
3. Create an inventory file specifying `localhost` as the target.
4. Create an HTML page.
5. Write a playbook to ensure the web directory exists and copy the HTML page into it.
6. Run the playbook and verify successful execution.
7. Install and start Apache, then open `http://localhost` to verify the page.

**Result:** The HTML website is deployed to `/var/www/html/index.html` using Ansible. When Apache is running, the page can be viewed in the browser.

## Troubleshooting

- **No Ubuntu distribution:** Run `wsl --install -d Ubuntu` in PowerShell as Administrator, then restart if prompted.
- **`ansible: command not found`:** Confirm commands are being run inside Ubuntu and run `sudo apt update` followed by `sudo apt install ansible -y`.
- **Permission denied:** Run the playbook with `--ask-become-pass`; ensure `become: true` is present.
- **Website does not load:** Run `sudo service apache2 status` and start Apache if necessary.
- **`code .` does not work:** Use `nano` to create/edit files, or install VS Code and its WSL extension.
