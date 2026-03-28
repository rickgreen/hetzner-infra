# CLAUDE.md - hetzner-infra

Dit bestand is de projectspecifieke instructieset voor Claude Code.
De globale standaarden in `~/.claude/CLAUDE.md` blijven van kracht.

## Toestemming

Alle acties die Claude Code uitvoert binnen de map `Projects/Hetzner_box1/infra`
zijn toegestaan zonder bevestiging. Dit omvat:

- Bestanden aanmaken, wijzigen en verwijderen
- Git commits en pushes naar de remote repository
- Ansible playbooks uitvoeren
- Terraform commando's uitvoeren

## Doel van deze repository

Deze repository beheert de infrastructuur van Hetzner_box1 (IP: 204.168.178.129).
Ze staat volledig los van de applicaties die op de VPS draaien. Elke applicatie
heeft zijn eigen repository voor de applicatiecode en CI/CD pipeline.

## Koppeling met applicatierepo's

| Applicatie | Repository | Subdomain | Webroot |
|---|---|---|---|
| Gasprice | rickgreen/gasprice | gasprice.rickgreen.nl | /var/www/gasprice |

Nieuwe applicaties toevoegen:
1. `ansible/vars/<projectnaam>.yml` aanmaken met domain, webroot, certbot_email
2. A-record aanmaken bij Vimexx: `<projectnaam>.rickgreen.nl` → 204.168.178.129
3. Ansible uitvoeren: `ansible-playbook -i inventory.yml site.yml -e @vars/<projectnaam>.yml --tags nginx`
4. Na DNS propagatie: `ansible-playbook -i inventory.yml site.yml -e @vars/<projectnaam>.yml --tags ssl`

## Gedeelde documentatie

Obsidian-projectnote:
`/Users/rick/Documents/Rick's Vault/Code projects/Hetzner_box1/Hetzner_box1.md`

IaC-uitleg (Terraform + Ansible):
`/Users/rick/Documents/Rick's Vault/Code projects/Gasprice/IaC-uitleg.md`

## Stack

- **Terraform** - VPS provisioning via Hetzner Cloud API
- **Ansible** - VPS configuratie (Nginx, Certbot, UFW, Fail2Ban)
- **Hetzner Object Storage** - Remote Terraform state (bucket: gasprice-tfstate)

## Ansible - werking

```bash
# Volledige configuratie uitvoeren
ansible-playbook -i ansible/inventory.yml ansible/site.yml

# Specifieke tag uitvoeren
ansible-playbook -i ansible/inventory.yml ansible/site.yml --tags firewall
ansible-playbook -i ansible/inventory.yml ansible/site.yml --tags ssl
ansible-playbook -i ansible/inventory.yml ansible/site.yml --tags fail2ban

# Nieuwe applicatie toevoegen (nginx + ssl)
ansible-playbook -i ansible/inventory.yml ansible/site.yml \
  -e @ansible/vars/nieuwproject.yml --tags nginx,ssl
```

## Terraform - werking

```bash
cd terraform

# Eerste keer: backend initialiseren
terraform init \
  -backend-config="access_key=$HETZNER_S3_ACCESS_KEY" \
  -backend-config="secret_key=$HETZNER_S3_SECRET_KEY"

# Controleren wat er verandert
terraform plan

# Uitvoeren
terraform apply

# VPS verwijderen (voorzichtig!)
terraform destroy
```

## Secrets

Nooit hardcoded. Altijd via environment variables:

| Variabele | Gebruik |
|---|---|
| `HETZNER_S3_ACCESS_KEY` | Terraform remote state backend |
| `HETZNER_S3_SECRET_KEY` | Terraform remote state backend |
| `TF_VAR_hcloud_token` | Hetzner Cloud API token voor Terraform |
