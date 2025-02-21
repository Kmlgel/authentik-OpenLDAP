# Authentik-OpenLDAP
Mise en place de la synchronisation des utilisateurs OpenLDAP à ceux de Authentik

Authentik LDAP
===


> [name=Kamal OILI koili@dawan.fr]

# Sommaire

[TOC]

![](https://hedgedoc.dawan.fr/uploads/upload_b51e7af8f7635349255fbcb3a29afd46.png)

# Prérequis

3 Machines : 

2 serveur : (ubuntu)

1 clients : (Windows)

# Préparer le ldap

* Installation mises à jour 

```bash=
sudo apt update && sudo apt upgrade -y
```
* Installer OpenLDAP et ses utils

```bash=
sudo apt install slapd ldap-utils -y
```

:::info
La création du domaine ce fait avec la commande suivante
:::

```bash=
sudo dpkg-reconfigure slapd
```
```ldif=
Omit OpenLDAP server configuration? → No
Domain Name → Ex: example.com (ce sera converti en dc=example,dc=com)
Organisation Name → Ex: Example Corp
Mot de passe LDAP Admin → Définissez un mot de passe sécurisé
Database backend → MDB
Remove old database? → Yes
Move old database? → Yes
Allow LDAPv2? → No
```
![](https://hedgedoc.dawan.fr/uploads/upload_ea2e220977c542bfdd2122309a6c0d23.png)

![](https://hedgedoc.dawan.fr/uploads/upload_c498e8d59dabb4dc86149eb5d8180232.png)

![](https://hedgedoc.dawan.fr/uploads/upload_aabbd9d0ff1576a76deeeac85a40e481.png)

![](https://hedgedoc.dawan.fr/uploads/upload_adb4da0f86be1905bd6b34b7bff5a530.png)

![](https://hedgedoc.dawan.fr/uploads/upload_8298b2a250752e9a8bc1ac1d34b6debe.png)

* Vérification du service 

```bash=
sudo systemctl status slapd
```
![](https://hedgedoc.dawan.fr/uploads/upload_7bf1e4a9c3c9d3e31bacaf28c894abfe.png)


* Pour persister le service au démarrage de la machine 

```bash=
sudo systemctl start slapd
```

## OpenLDAP prêt => Configuration 2 solutions

- Ligne de commande :x: 
- utilitaire :ballot_box_with_check: (ldapadmin)

* Connexion avec a ldapadmin à OpenLDAP

![](https://hedgedoc.dawan.fr/uploads/upload_19c1ba7f1f1c0f6aa540bac969b0d98d.png)

* Création des utilisateurs de l'annuaire

![](https://hedgedoc.dawan.fr/uploads/upload_33696096a1ba4b73e13f5183be2693a7.png)

![](https://hedgedoc.dawan.fr/uploads/upload_5f93678cab602e8586819a93ec4991a3.png)


# Préparation d'authentik (Docker)

* Installation docker/docker compose

* copier le compose dans un fichier (wget https://goauthentik.io/docker-compose.yml)

* Créer un fichier .env pour les variables nécessaire au bon fonctionnement de l'application

* Lancement de la stack depuis le dossier ou ce trouve le compose

```bash=
docker compose up -d 
```

:::info
lors de la première connexion l'utilisateur par défaut/admin n'est pas créer utiliser ce lien afin de l'initialiser
http://<your server's IP or hostname>:9000/if/flow/initial-setup/
:::

* Connexion à l'utilisateur admin 

## Configuration de la synchronisation

* Utilisateurs Openldap

![](https://hedgedoc.dawan.fr/uploads/upload_0eb45abac392fce10902c8f622ba3032.png)

* Utilisateurs Authentik 

![](https://hedgedoc.dawan.fr/uploads/upload_3abdcb0f1d79efc8fc07c3ec905c86fc.png)

* Se rendre dans la partie "admin interface" => "Directory" => "Federation and Social login"

* Cliquer sur créer => "LDAP source"

![](https://hedgedoc.dawan.fr/uploads/upload_55f6cc10e9dd5869befd6b99c28ce596.png)

* Remplir le champ Name sa complétera directement le slug

![](https://hedgedoc.dawan.fr/uploads/upload_f24b1567386adef23b45d88c781bdffe.png)

:::info
l'option "upate internal password on login" permet de mettre a jour le mot de passe lors de la synchronisation si celui en backend a changer avant une synchronisation
:::

* ajout de l'URI d'accès au ldap

![](https://hedgedoc.dawan.fr/uploads/upload_4b365bb447f69ddd0cedac0eba0d7ecc.png)

![](https://hedgedoc.dawan.fr/uploads/upload_933615c4a2634dfd3d0cd50c54e3d528.png)

* les "property user" et "property group" vérifier que la partie "OpenLDAP mapping" est bien des deux cotés sinon selectionner et basculer dans la partie "selected"

![](https://hedgedoc.dawan.fr/uploads/upload_25a3b9a4230de1c283691dcc94a19781.png)

exemple 

![](https://hedgedoc.dawan.fr/uploads/upload_f06c6027de84914ed0c0b644dcab5fb2.png)

![](https://hedgedoc.dawan.fr/uploads/upload_4b70c256766594e8997beeca84c309de.png)

* Par défaut 

![](https://hedgedoc.dawan.fr/uploads/upload_5cedf73121bd5ddbaac94d126b4d05e2.png)

* Afin d'avoir ces informations la

```bash
ldapsearch -x -H ldap://[IP_SERVEUR_LDAP]-D "CN=admin,DC=formation,DC=lan" -W -b "DC=formation,DC=lan"
image.png
```
![](https://hedgedoc.dawan.fr/uploads/upload_ea634462bfc08d9da521ca57d9ce8cba.png)

![](https://hedgedoc.dawan.fr/uploads/upload_5f355f9208b3a949030d2971b6310528.png)

![](https://hedgedoc.dawan.fr/uploads/upload_c815c2566fd4e1d6b64698f5704631bc.png)

:::info
pourquoi "cn" ?
:::

![](https://hedgedoc.dawan.fr/uploads/upload_e1ca0107bcf0f69db1b6135e46d4b3eb.PNG)

![](https://hedgedoc.dawan.fr/uploads/upload_262ae3337ee546701397dabb5b1c51a7.png)

### Forcer la synchronisation 

* Commande a éxecuter depuis la machine authentik 

```ldif=
docker exec -it authentik-server-1 ak ldap_sync slugladp
```
![](https://hedgedoc.dawan.fr/uploads/upload_86331221996d54a4bd7970a0d420429d.png)

![](https://hedgedoc.dawan.fr/uploads/upload_357033b91330f1ad0a48ae391059b2ef.png)

:::warning
lors de votre synchronisation si cela se termine par "IndentationError: unindent does not match any outer indentation level""
:::
* Modifier/vérifier ce fichier si il n'y a pas de problème d'indentation 

![](https://hedgedoc.dawan.fr/uploads/upload_4f890cd4b8c1c58b1587363176838448.png)

* Corriger

```python=
from authentik.lib.utils.dict import set_path_in_dict

def get_field(dn, source):
    path_elements = []
    for pair in dn.split(","):
        attr, _, value = pair.partition("=")
        # Ignore elements from the Root DSE and the canonical name of the object
        if attr.lower() in ["cn", "dc"]:
            continue
        path_elements.append(value)
    
    path_elements.reverse()
    path = source.get_user_path()
    
    if path_elements:
        path = f"{path}/{'/'.join(path_elements)}"
    
    return path

def process_ldap_mapping(dn, source):
    field = "path"
    result = {"attributes": {}}
    
    if field.startswith("attributes."):
        set_path_in_dict(result, field, get_field(dn, source), sep=".")
    else:
        result[field] = get_field(dn, source)
    
    return result
```

* Vérification que tous les utilisateur sont arriver sur authentik

![](https://hedgedoc.dawan.fr/uploads/upload_3996731d37cbbbc39c31cc2fa6e6a617.png)

![](https://hedgedoc.dawan.fr/uploads/upload_5d8f9d63c0f35ef9e0c69e8449b1aac9.png)

# TEST connexion

![](https://hedgedoc.dawan.fr/uploads/upload_07f0a9c5bcb2a904adde552328958e06.png)

![](https://hedgedoc.dawan.fr/uploads/upload_775fb674d204f3b25148bd4c022c5f29.png)
