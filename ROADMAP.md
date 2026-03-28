# ROADMAP.md - hetzner-infra

Status: [ ] = open, [x] = klaar.

---

## Opgezet ✓

- [x] Mapstructuur: `Hetzner_box1/infra/` als eigen repository
- [x] Verplaatst vanuit `gasprice/` naar eigen repo (2026-03-28)
- [x] GitHub repository aanmaken: `rickgreen/hetzner-infra`

## Terraform ✓

- [x] `terraform/main.tf` - Hetzner provider, VPS resource (CX23, Ubuntu 24.04, nbg1)
- [x] `terraform/variables.tf`, `outputs.tf`, `versions.tf`
- [x] `terraform/terraform.tfvars.example`
- [x] Remote state in Hetzner Object Storage (bucket: `gasprice-tfstate`)

## Ansible ✓

- [x] `ansible/roles/ufw/` - UFW firewall
- [x] `ansible/roles/fail2ban/` - SSH + Nginx jails
- [x] `ansible/roles/hardening/` - unattended-upgrades verificatie
- [x] `ansible/roles/deploy_user/` - deploy-gebruiker, SSH key, webroot
- [x] `ansible/roles/nginx/` - Nginx vhost templates
- [x] `ansible/roles/certbot/` - Let's Encrypt SSL, automatische verlenging
- [x] `ansible/vars/gasprice.yml` - variabelen voor Gasprice
- [x] Playbook succesvol uitgevoerd: 30 taken, 0 mislukt

## Actieve projecten op de VPS

| Project | Subdomain | Webroot | Vars-file |
|---|---|---|---|
| Gasprice | gasprice.rickgreen.nl | /var/www/gasprice | ansible/vars/gasprice.yml |

---

## Nieuw project toevoegen (checklist)

- [ ] A-record aanmaken bij Vimexx: `<naam>.rickgreen.nl` → `204.168.178.129`
- [ ] `ansible/vars/<naam>.yml` aanmaken
- [ ] Nginx vhost aanmaken: `ansible-playbook ... -e @vars/<naam>.yml --tags nginx`
- [ ] Wachten op DNS propagatie
- [ ] SSL aanvragen: `ansible-playbook ... -e @vars/<naam>.yml --tags ssl`
- [ ] Verifiëren: HTTPS bereikbaar, HSTS header aanwezig, HTTP redirect werkt
- [ ] Tabel 'Actieve projecten' hierboven bijwerken
