## Deploy a vpn server to Digital Ocean

In conf, have:
(or create by following README.ssl)
ca.crt ca.key dh1024.pem
id_rsa
server.crt server.csr server.key

## Deploy stage vpnbox to digital ocean manually

create droplet
image CentOS 7.x x64
hostname vpnbox-stage.phu73l.net
size smallest
region San Francisco 1
ssh key (personal key)

create or replace A record for new vpnbox-stage
wait for DNS to propagate

create or check out release branch

deploy stage:

  ansible-playbook -i hosts playbook.yml --vault-password-file=conf/vault_pass.txt

test:

  verify that traffic for client to vpnbox_stage goes through vpnbox_stage
  view connected clients in /etc/openvpn/openvpn-status.log

## promote stage to prod

There must be at least one prod vpn server running:
vpnbox-prod-foo.phu73l.net
vpnbox-prod-bar.phu73l.net
We will promote stage to the one not currently running, and then decomission the currently running one.

rename vpnbox-stage droplet to new vpnbox-prod-foo|bar.phu73l.net
rename vpnbox-stage hostname to new vpnbox-prod-foo|bar.phu73l.net
  ssh -t -F local/ssh_config vpnbox-stage.phu73l.net 'sudo hostnamectl set-hostname vpnbox-prod-foo.phu73l.net'
change A record for vpnbox-stage to point to new vpnbox-prod-foo|bar.phu73l.net
remove A record for vpnbox-stage
wait for DNS to propagate
refresh iptables on asteriskserver prod, in futel-installation repo
XXX is this necessary if the vpnbox hostname is the FQDN?
  ansible-playbook -i hosts update_iptables_playbook.yml
stop openvpn on old vpnbox-prod-foo|bar.phu73l.net
  sudo service openvpn stop
XXX wait 1-30 minutes for sip peers to become reachable?
test that new vpnbox-prod-foo|bar.phu73l.net is being used
  sip show peers on futel-prod
  service openvpn@server status on new vpnbox-prod-foo|bar.phu73l.net
  cat /etc/openvpn/openvpn-status.log on new vpnbox-prod-foo|bar.phu73l.net
make a snapshot of old vpnbox-prod-foo|bar.phu73l.net
destroy droplet old vpnbox-prod-foo|bar.phu73l.net
remove A record for vpnbox-prod-foo|bar.phu73l.net
remove snapshots of vpnbox-prod-foo|bar.phu73l.net except for most recent
