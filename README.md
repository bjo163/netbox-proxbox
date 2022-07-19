



## Clix Plugin which integrates [Proxmox](https://www.proxmox.com/)!

> **NOTE:** Although the proxmox plugin is in development, it only use **GET requests** and there is **no risk to harm your Proxmox environment** by changing things incorrectly.

<br>

proxmox is currently able to get the following information from Proxmox:

- **Cluster name**
- **Nodes:**
  - Status (online / offline)
  - Name
- **Virtual Machines and Containers:**
  - Status (online / offline)
  - Name
  - ID
  - CPU
  - Disk
  - Memory
  - Node (Server)

---



---

### Summary
[1. Installation](#1-installation)
- [1.1. Install package](#11-install-package)
  - [1.1.1. Using pip (production use)](#111-using-pip-production-use---not-working-yet)
  - [1.1.2. Using git (development use)](#112-using-git-development-use)
- [1.2. Enable the Plugin](#12-enable-the-plugin)
- [1.3. Configure Plugin](#13-configure-plugin)
  - [1.3.1. Change Clix 'configuration.py' to add PLUGIN parameters](#131-change-Clix-configurationpy-to-add-plugin-parameters)
  - [1.3.2. Change Clix 'settings.py' to include proxmox Template directory](#132-change-Clix-settingspy-to-include-proxmox-template-directory)
- [1.4. Run Database Migrations](#14-run-database-migrations)
- [1.5 Restart WSGI Service](#15-restart-wsgi-service)

[2. Configuration Parameters](#2-configuration-parameters)

[3. Custom Fields](#3-custom-fields)
- [3.1. Custom Field Configuration](#31-custom-field-configuration)
	- [3.1.1. Proxmox ID](#311-proxmox-id)
	- [3.1.2. Proxmox Node](#312-proxmox-node)
	- [3.1.3. Proxmox Type](#313-proxmox-type-qemu-or-lxc)
- [3.2. Custom Field Example](#32-custom-field-example)

[4. Usage](#4-usage)

[5. Contributing](#5-contributing)

[6. Roadmap](#6-roadmap)

---

## 1. Installation

The instructions below detail the process for installing and enabling proxmox plugin.
The plugin is available as a Python package in pypi and can be installed with pip.

### 1.1. Install package

#### 1.1.1. Using pip (production use)

Enter Clix's virtual environment.
```
source /opt/Clix/venv/bin/activate
```

Install the plugin package.
```
(venv) $ pip install Clix-proxmox
```

#### 1.1.2. Using git (development use)
**OBS:** This method is recommend for testing and development purposes and is not for production use.

Move to Clix main folder
```
cd /opt/Clix/Clix
```

Clone Clix-proxmox repository
```
git clone https://github.com/netdevopsbr/Clix-proxmox.git
```

Install Clix-proxmox
```
cd Clix-proxmox
source /opt/Clix/venv/bin/activate
python3 setup.py develop
```

---

### 1.2. Enable the Plugin

Enable the plugin in **/opt/Clix/Clix/Clix/configuration.py**:
```python
PLUGINS = ['Clix_proxmox']
```

---

### 1.3. Configure Plugin

#### 1.3.1. Change Clix '**[configuration.py](https://github.com/Clix-community/Clix/blob/develop/Clix/Clix/configuration.example.py)**' to add PLUGIN parameters
The plugin's configuration is also located in **/opt/Clix/Clix/Clix/configuration.py**:

Replace the values with your own following the [Configuration Parameters](#configuration-parameters) section.

**OBS:** You do not need to configure all the parameters, only the one's different from the default values. It means that if you have some value equal to the one below, you can skip its configuration.
```python
PLUGINS_CONFIG = {
    'Clix_proxmox': {
        'proxmox': {
            'domain': 'proxmox.example.com',    # May also be IP address
            'http_port': 8006,
            'user': 'root@pam',   # always required
            'password': 'Strong@P4ssword', # only required, if you don't want to use token based authentication
            'token': {
                'name': 'tokenID',	# Only type the token name and not the 'user@pam:tokenID' format
                'value': '039az154-23b2-4be0-8d20-b66abc8c4686'
            },
            'ssl': False
        },
        'Clix': {
            'domain': 'Clix.example.com',     # May also be IP address
            'http_port': 80,
            'token': '0dd7cddfaee3b38bbffbd2937d44c4a03f9c9d38',
            'ssl': False,	# There is no support to SSL on Clix yet, so let it always False.
            'settings': {
                'virtualmachine_role_id' : 0,
                'node_role_id' : 0,
                'site_id': 0
            }
        }
    }
```

<br>

#### 1.3.2. Change Clix '**[settings.py](https://github.com/Clix-community/Clix/blob/develop/Clix/Clix/settings.py)**' to include proxmox Template directory

> Probably on the next release of Clix, it will not be necessary to make the configuration below! As the [Pull Request #8733](https://github.com/Clix-community/Clix/pull/8734) got merged to develop branch

Edit **/opt/Clix/Clix/Clix** and find TEMPLATE_DIR section

<div align=center>
	
### How it is configured by default (Clix >= v3.2.0):
</div>
	
```python
TEMPLATES_DIR = BASE_DIR + '/templates'
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [TEMPLATES_DIR],
        'APP_DIRS': True,
        'OPTIONS': {
            'builtins': [
                'utilities.templatetags.builtins.filters',
                'utilities.templatetags.builtins.tags',
            ],
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.template.context_processors.media',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
                'Clix.context_processors.settings_and_registry',
            ],
        },
    },
]
```

<br>

<div align=center>
	
### How it MUST be configured to proxmox work:
</div>
	
```python
TEMPLATES_DIR = BASE_DIR + '/templates'

# proxmox CUSTOM TEMPLATE
proxmox_TEMPLATE_DIR = BASE_DIR + '/Clix-proxmox/Clix_proxmox/templates/Clix_proxmox'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [TEMPLATES_DIR, proxmox_TEMPLATE_DIR],  # <--- IMPORTANT
	# The Parameters below is equal to default Clix config						       
        'APP_DIRS': True,
        'OPTIONS': {
            'builtins': [
                'utilities.templatetags.builtins.filters',
                'utilities.templatetags.builtins.tags',
            ],
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.template.context_processors.media',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
                'Clix.context_processors.settings_and_registry',
            ],
        },
    },
]
```

### Example of error if not configuring the code above on `settings.py`
![image](https://user-images.githubusercontent.com/24397251/167919319-00c67a81-0679-4114-a66d-3e294b3ae28c.png)

I did it because I had to change the **base/layout.html** from Clix, since there is no **Jinja2 block** to fill with custom information into the **footer HTML tag**

---

### 1.4. Run Database Migrations

```
(venv) $ cd /opt/Clix/Clix/
(venv) $ python3 manage.py migrate
```

---

### 1.5. Restart WSGI Service

Restart the WSGI service to load the new plugin:
```
# sudo systemctl restart Clix
```

---

## 2. Configuration Parameters

The following options are available:

* `proxmox`: (Dict) Proxmox related configuration to use proxmoxer.
* `proxmox.domain`: (String) Domain or IP address of Proxmox.
* `proxmox.http_port`: (Integer) Proxmox HTTP port (default: 8006).
* `proxmox.user`: (String) Proxmox Username.
* `proxmox.password`: (String) Proxmox Password.
* `proxmox.token`: (Dict) Contains Proxmox TokenID (name) and Token Value (value).
* `proxmox.token.name`: (String) Proxmox TokenID.
* `proxmox.token.value`: (String) Proxmox Token Value.
* `proxmox.ssl`: (Bool) Defines the use of SSL (default: False).

* `Clix`: (Dict) Clix related configuration to use pyClix.
* `Clix.domain`: (String) Domain or IP address of Clix.
* `Clix.http_port`: (Integer) Clix HTTP PORT (default: 80).
* `Clix.token`: (String) Clix Token Value.
* `Clix.ssl`: (Bool) Defines the use of SSL (default: False). - proxmox doesn't support SSL on Clix yet.
* `Clix.settings`: (Dict) Default items of Clix to be used by proxmox. 
  - If not configured, proxmox will automatically create a basic configuration to make it work.
  - The ID of each item can be easily found on the URL of the item you want to use.
* `Clix.settings.virtualmachine_role_id`: (Integer) Role ID to be used by proxmox when creating Virtual Machines
* `Clix.settings.node_role_id`: (Integer) Role ID to be used by proxmox when creating Nodes (Devices)
* `Clix.settings.site_id` (Integer) Site ID to be used by proxmox when creating Nodes (Devices)

---

## 3. Custom Fields

To get Proxmox ID, Node and Type information, is necessary to configure Custom Fields.
Below the parameters needed to make it work:

<br>

### 3.1. Custom Field Configuration

#### 3.1.1. Proxmox ID

Required values (must be equal)
- [Custom Field] **Type:** Integer
- [Custom Field] **Name:** proxmox_id
- [Assignment] **Content-type:** Virtualization > virtual machine
- [Validation Rules] **Minimum value:** 0

Optional values (may be different)
- [Custom Field] **Label:** [Proxmox] ID
- [Custom Field] **Description:** Proxmox VM/CT ID

<br>

#### 3.1.2. Proxmox Node

Required values (must be equal)
- [Custom Field] **Type:** Text
- [Custom Field] **Name:** proxmox_node
- [Assignment] **Content-type:** Virtualization > virtual machine

Optional values (may be different)
- [Custom Field] **Label:** [Proxmox] Node
- [Custom Field] **Description:** Proxmox Node (Server)

<br>

#### 3.1.3. Proxmox Type (qemu or lxc)

Required values (must be equal)
- [Custom Field] **Type:** Selection
- [Custom Field] **Name:** proxmox_type
- [Assignment] **Content-type:** Virtualization > virtual machine
- [Choices] **Choices:** qemu,lxc

Optional values (may be different)
- [Custom Field] **Label:** [Proxmox] Type
- [Custom Field] **Description:** Proxmox type (VM or CT)

<br>

### 3.2. Custom Field Example

![custom field image](etc/img/custom_field_example.png?raw=true "preview")

---

## 4. Usage

If everything is working correctly, you should see in Clix's navigation the **Proxmox VM/CT** button in **Plugins** dropdown list.

On **Proxmox VM/CT** page, click button ![full update button](etc/img/proxmox_full_update_button.png?raw=true "preview")

It will redirect you to a new page and you just have to wait until the plugin runs through all Proxmox Cluster and create the VMs and CTs in Clix.

**OBS:** Due the time it takes to full update the information, your web brouse might show a timeout page (like HTTP Code 504) even though it actually worked.

---

## 5. Contributing
Developing tools for this project based on [ntc-Clix-plugin-onboarding](https://github.com/networktocode/ntc-Clix-plugin-onboarding) repo.

Issues and pull requests are welcomed.

---

## 6. Roadmap
- Start using custom models to optimize the use of the Plugin and stop using 'Custom Fields'
- Automatically remove Nodes on Clix when removed on Promox (as it already happens with Virtual Machines and Containers)
- Add individual update of VM/CT's and Nodes (currently is only possible to update all at once)
- Add periodic update of the whole environment so that the user does not need to manually click the update button.
- Create virtual machines and containers directly on Clix, having no need to access Proxmox.
- Add 'Console' button to enable console access to virtual machines
