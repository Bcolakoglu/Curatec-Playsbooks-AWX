# 🧾 Ansible Playbook Cheatsheet for AWX

Een praktische naslaggids voor het schrijven van Ansible playbooks binnen AWX. Dit document bevat best practices, veelgebruikte modules, tags en tips specifiek gericht op gebruik binnen AWX.

---

## 📁 Structuur van een Basis Playbook

```yaml
---
- name: Voorbeeld playbook
  hosts: all
  become: true
  tasks:
    - name: Installeer pakket
      ansible.builtin.apt:
        name: htop
        state: present
      tags:
        - install
```

---

## 🏷️ Veelgebruikte Tags

| Tag         | Beschrijving                              |
|-------------|--------------------------------------------|
| `install`   | Voor installatie van pakketten             |
| `config`    | Configuratiebestanden aanpassen            |
| `service`   | Starten/herstarten van services            |
| `restart`   | Alleen taken die services herstarten       |
| `check`     | Validatie- of verificatietaken             |
| `debug`     | Debug of output-taken                      |
| `health`    | Status/gezondheid van systeem/applicatie   |
| `files`     | Taken die bestanden manipuleren            |

---

## 🔧 Veelgebruikte Modules

| Module               | Doel                                     |
|----------------------|------------------------------------------|
| `apt`                | Pakketten installeren op Debian/Ubuntu   |
| `yum` / `dnf`        | Pakketten installeren op RHEL/CentOS     |
| `copy`               | Kopieert bestanden                       |
| `template`           | Gebruikt Jinja2-templates                |
| `lineinfile`         | Bewerk een enkele regel in een bestand   |
| `systemd`            | Start/stop/restart services              |
| `command` / `shell`  | Voert shell-commando's uit               |
| `debug`              | Print variabelen of tekst                |
| `apt_repository`     | Repo's toevoegen                         |
| `apt_key`            | GPG keys toevoegen                       |

---

## 🧪 Dry-run (Check Mode)

### In YAML:
```yaml
check_mode: yes
```

### In AWX via Extra Vars:
```yaml
ansible_check_mode: true
```

Let op: niet alle modules ondersteunen check mode volledig.

---

## 🔁 Handlers en Notify

```yaml
tasks:
  - name: Wijzig config
    ansible.builtin.lineinfile:
      path: /etc/app.conf
      regexp: '^Option='
      line: 'Option=yes'
    notify: Restart app

handlers:
  - name: Restart app
    ansible.builtin.systemd:
      name: app
      state: restarted
```

---

## 🧠 Tips voor AWX

- Gebruik **Inventories** om hosts te groeperen
- Beheer secrets via **Credentials**
- Gebruik **Job Templates** met Extra Vars voor flexibiliteit
- Versiebeheer je playbooks via een **Git Project**
- Gebruik `tags` om specifieke delen van playbooks te draaien:
  ```bash
  ansible-playbook site.yml --tags "install,config"
  ```

---

## 📄 Voorbeeld van een Zabbix Agent 2 Playbook

```yaml
- name: Installeer Zabbix Agent 2
  hosts: all
  become: true
  vars:
    zabbix_server_ip: "10.0.0.100"

  tasks:
    - name: Installeer agent
      ansible.builtin.apt:
        name: zabbix-agent2
        state: present
      tags:
        - install

    - name: Configureer agent
      ansible.builtin.lineinfile:
        path: /etc/zabbix/zabbix_agent2.conf
        regexp: '^Server='
        line: "Server={{ zabbix_server_ip }}"
      tags:
        - config

    - name: Start de agent
      ansible.builtin.systemd:
        name: zabbix-agent2
        enabled: yes
        state: started
      tags:
        - service
```

---

## 📚 Extra

- Gebruik `gather_facts: false` voor snellere playbooks als facts niet nodig zijn
- Gebruik `backup: yes` in `lineinfile`, `template`, etc. om wijzigingen veilig te maken
- Combineer `--check` en `--diff` om te zien wat er zou veranderen

---

## 🔁 Verschil tussen Tasks en Handlers

| Kenmerk       | **Tasks**                               | **Handlers**                            |
|---------------|-----------------------------------------|-----------------------------------------|
| **Wanneer uitgevoerd?** | Altijd, tenzij `when` of `tags` zegt van niet | Alleen als ze "aangeroepen" worden door `notify` |
| **Hoe vaak uitgevoerd?** | Elke keer dat de playbook draait (mits van toepassing) | Slechts **één keer per play**, en **pas aan het einde** |
| **Waarvoor bedoeld?** | Alle standaardacties: pakketten, configs, services, etc. | Acties die **moeten gebeuren ná een wijziging** (bijv. herstarten) |
| **Gebruiksscenario** | Alles van installeren tot configureren | Restart van een service na config-wijziging |
| **Declaratieplek** | Onder `tasks:` | Onder `handlers:` |

### 🔁 Voorbeeld

```yaml
tasks:
  - name: Wijzig configuratie
    ansible.builtin.lineinfile:
      path: /etc/demo.conf
      regexp: '^enabled='
      line: 'enabled=true'
    notify: Restart demo service

handlers:
  - name: Restart demo service
    ansible.builtin.systemd:
      name: demo
      state: restarted
```

🧠 In dit voorbeeld:
- De **task** verandert het configbestand
- Als er iets gewijzigd is, wordt de **handler** "aangeroepen" via `notify`
- De handler voert zijn taak **pas aan het einde van de play** uit, en **slechts één keer** zelfs als meerdere taken hem zouden aanroepen

---

*Laatste update: Augustus 2025*
