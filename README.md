# Projet infra
## Routeur
Pour répondre a la demande nous avons mis en place une infrastructure avec deux LAN. le LAN1 (192.168.3.1/24) comportant le réseau interne de l'entreprise, un vpn et un serveur de fichier interne a l'entreprise puis un portail captif pour permettre que seul les utilisateurs authentifié puisse aller sur l'internet. Le LAN2 (192.168.4.1/24) Comportant le site web de l'entreprise. Pour le router nous avons choisi PfSense sur lequel il y a HAproxy pour permettre de faire du load balancing sur les serveurs web de l'entreprise et un un reverse proxy.
## Schéma d'infra
![picture](/image/infra.png)
## Plan d'addressage
|Machine|LAN1|LAN2|
|-------|----|----|
|File server	|192.168.3.10	 |	X  |
|VPN		|192.168.3.11	 |	X  |
|Employé		|192.168.3.100-245	 |	X  |
|Web01		| X |192.168.4.10  |
|Web02		| X |192.168.4.11  |
|NFS web		| X |192.168.4.50  |

## VPN
Nous avons d'abord installé pipvpn sur une Raspberry, puis, sur une suggestion de Léo, nous nous sommes tournés vers Wireguard. Nouveau VPN ***encore en développement***, Wireguard offre une connexion plus rapide, avec moins de latence.
### Installation côté serveur
[https://www.ckn.io/blog/2017/11/14/wireguard-vpn-typical-setup/](Selon ce guide)

Après avoir ajouté le repository pour pouvoir installer Wireguard, et après l'avoir installé, il faut ouvrir les ports pour accepter les connexions sur le port choisi (pour nous 1424). On active aussi le forwarding (redirection) vers l'interface réseau que l'on souhaite. Dans notre cas, on redirige vers l'interface qui correspond au LAN1, pour permettre aux employés d'accéder au LAN1 à travers le VPN. On crée ensuite le fichier de configuration du serveur dans `/etc/wireguard/wg0-server.conf` :

```
[Interface]
Address = 10.9.13.1/24
SaveConfig = true
PrivateKey = <insert server_private_key>
ListenPort = 1424
```

Les clés sont générées par la commande `wg genkey | tee server_private_key | wg pubkey > server_public_key` en base64.
On peut ensuite allumer cette interface avec la commande `sudo wg-quick up wg0-server` (wg0-server est le nom de la configuration que l'on veut utiliser).
Il est ensuite possible d'ajouter un client avec les commandes suivantes :
```
wg genkey | tee client_private_key | wg pubkey > client_public_key
sudo wg set wg0-server peer $(cat client_public_key) allowed-ips <new_client_vpn_IP>/32
```
Cependant, il faut aussi générer un fichier de configuration correspondant pour le client, ce pourquoi nous avons réalisé un script.
### Installation côté client
De même, après avoir ajouté le repository pour pouvoir installer Wireguard, et après l'avoir installé, il suffit de rajouter le fichier de configuration généré par le serveur dans `/etc/wireguard/wg0-client.conf`, puis d'exécuter la commande `sudo wg-quick up wg0-client`.
