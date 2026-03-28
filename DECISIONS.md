# DECISIONS.md - hetzner-infra

Architectuurkeuzes met datum en motivatie.

---

## 2026-03-28 - Infra als aparte repository, los van applicaties

**Keuze:** Terraform en Ansible staan in een eigen repository (`hetzner-infra`),
gescheiden van de applicatierepo's die op de VPS draaien.

**Motivatie:** Infrastructuur en applicaties hebben verschillende verantwoordelijkheden
en verschillende change-frequenties. De Gasprice-app wordt bij elke code-wijziging
gedeployed; de VPS-configuratie verandert zelden. Een developer die aan Gasprice
werkt hoeft niets te weten van Ansible. De infra-repo beheert de VPS als geheel,
niet één specifieke applicatie.

**Bewust niet gekozen:** Ansible/Terraform onderdeel maken van elke applicatierepo
(leidt tot duplicatie en koppeling), monorepo met alle projecten samen (te groot,
onduidelijke eigenaarschap).

---

## 2026-03-28 - Mapstructuur: Hetzner_box1/

**Keuze:** Onder `Projects/` een map `Hetzner_box1/` met daarin `gasprice/`
en `infra/` als submappen.

**Motivatie:** De mapstructuur weerspiegelt de fysieke werkelijkheid: één VPS,
meerdere projecten die erop draaien. Als er ooit een tweede Hetzner-machine
bijkomt, staat die logisch als `Hetzner_box2/` naast de eerste.

---

## 2026-03-28 - Geen GitHub Actions voor Ansible en Terraform

**Keuze:** Ansible en Terraform worden handmatig uitgevoerd via de CLI,
niet via GitHub Actions workflows.

**Motivatie:** Provisioning en configuratie zijn eenmalige of zeldzame operaties.
Automatisering via GitHub Actions voegt complexiteit toe (secrets beheren voor
Hetzner API, Terraform state locking) zonder proportionele meerwaarde voor
een setup van één VPS. De CLI geeft directe controle en zichtbare output.
Dit kan in een later stadium worden herzien als de schaal toeneemt.

---

## 2026-03-28 - Terraform state in Hetzner Object Storage

**Keuze:** Remote state opgeslagen in Hetzner Object Storage (S3-compatibel,
bucket `gasprice-tfstate`, regio Falkenstein).

**Motivatie:** Terraform state bevat gevoelige informatie (IP-adressen,
resource-IDs) en mag nooit in Git. Lokale state is een risico bij verlies
van de werkplek. Hetzner Object Storage is Europees, valt binnen het
data-sovereignty principe, en is S3-compatibel zodat de standaard
Terraform S3 backend werkt.

---

## 2026-03-28 - jail.local boven jail.conf voor Fail2Ban

**Keuze:** Fail2Ban-configuratie gaat uitsluitend in `/etc/fail2ban/jail.local`,
nooit in `jail.conf`.

**Motivatie:** Pakketupdates overschrijven `jail.conf`. Wijzigingen in `jail.local`
blijven altijd behouden, ook na `apt upgrade`. Dit is de door Fail2Ban zelf
aanbevolen aanpak.

---

## 2026-03-28 - Één deploy-gebruiker voor alle projecten op de VPS

**Keuze:** GitHub Actions gebruikt voor alle projecten dezelfde `deploy`-gebruiker
met de `gasprice_deploy` SSH-key.

**Motivatie:** Bij een kleine VPS met persoonlijke projecten is één deploy-gebruiker
voldoende. Elke applicatie deployt naar zijn eigen webroot (`/var/www/<project>`),
dus er is geen risico op interferentie. Bij grotere setups met meerdere teams zou
je per project een aparte deploy-gebruiker overwegen.
