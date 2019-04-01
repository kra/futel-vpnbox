## Deploy a vpn server to Digital Ocean

In conf, have:
(or create by following README.ssl)
ca.crt ca.key dh1024.pem
id_rsa
server.crt server.csr server.key

# ## Deploy stage vpnbox to digital ocean manually

# create droplet
# image CentOS 7.x x64
# hostname vpnbox-stage.phu73l.net
# size smallest
# region San Francisco 1
# ssh key (personal key)

# create or replace A record for new vpnbox-stage
# wait for DNS to propagate

create droplet and subdomain:

create or check out release branch

  ansible-playbook -i deploy/hosts deploy_digitalocean_playbook.yml  --vault-password-file=conf/vault_pass.txt

wait for DNS to propagate with "nslookup vpnbox-stage.phu73l.net"

deploy stage:

  ansible-playbook -i deploy/hosts playbook.yml --vault-password-file=conf/vault_pass.txt

test:

  Verify that service runs on server:  
  systemctl status openvpn@server.service
  Verify that local box can contact TCP server:
  nmap -Pn -p 1194 vpnbox-stage.phu73l.net
  Verify that local box can contact UDP server:  
  nmap -Pn -sU -p 1194 vpnbox-stage.phu73l.net
  Verify that client works:
  set up client as directed in futel-vpnclient project
  verify that traffic for client to vpnbox_stage goes through vpnbox_stage
  view connected clients on server in /etc/openvpn/openvpn-status.log

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
  ansible-playbook -i deploy/hosts update_iptables_playbook.yml
stop openvpn on old vpnbox-prod-foo|bar.phu73l.net
  sudo service openvpn@server stop
  sudo service openvpn@server-tcp stop  
XXX wait 1-30 minutes for sip peers to become reachable?
test that new vpnbox-prod-foo|bar.phu73l.net is being used
  sip show peers on futel-prod
  service openvpn@server status on new vpnbox-prod-foo|bar.phu73l.net
  service openvpn@server-tcp status on new vpnbox-prod-foo|bar.phu73l.net  
  cat /etc/openvpn/openvpn-status.log on new vpnbox-prod-foo|bar.phu73l.net
  cat /etc/openvpn/openvpn-status-tcp.log on new vpnbox-prod-foo|bar.phu73l.net  
make a snapshot of old vpnbox-prod-foo|bar.phu73l.net
destroy droplet old vpnbox-prod-foo|bar.phu73l.net
remove A record for vpnbox-prod-foo|bar.phu73l.net
remove snapshots of vpnbox-prod-foo|bar.phu73l.net except for most recent

XXX install futel-substation on new prod box

## monitor prod

  ssh -F local/ssh_config vpnbox-prod-bar.phu73l.net
  view connected clients in /etc/openvpn/openvpn-status.log
