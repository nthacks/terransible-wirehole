# Wirehole on Hetzner Cloud

Terraform for deploying the Wirehole deployment onto Hetzner. If you choose the default server, it should cost < $5 USD a month and give you 20TB of transfer.

# Deploying Wirehole to Hetzner Cloud

* Have a Hetzner Cloud account and gather the following information:
  * Hetzner Cloud API Key
  * The name of an SSH Key loaded into Hetzner Cloud

# Deploy Wirehole

1. In the cloned `terransible-wirehole` directory, `cd ./terraform/hetzner`
2. Copy the `terraform.tfvars.example` file to `terraform.tfvars`
3. Update **ALL** of the values above the Optional section at the bottom
4. Run `terraform plan` and if this completes without any errors, proceed to the next step
5. Run `terraform apply --auto-approve` and wait for the deployment to finish. When it is complete you should see a new `hcloud-inventory` file in the `ansible/inventory` directory
6. Change into the `ansible` directory by running `cd ../../ansible`
7. Copy the `ansible.cfg.example` to `ansible.cfg`. The default values provided will work, but feel free to update to your liking if you are familiar with Ansible
8. Run `ansible ansible -m ping` and make sure it returns a `pong` confirming your connectivity. If there are no errors, move the the next step.
9. Run `anible-playbook hcloud_main.yml` and it will run through all of the provisioning steps and reboot the Wirehole host once to install updates.
10. If everything went well, look in the `wg_client_configs/hcloud` directory and you should find 10 configuration files for 10 Wireguard peers (Peer1 - Peer10)
11. Install Wireguard on your desired devices: https://www.wireguard.com/install/
12. Copy the config from `Peer1/peer1.conf` to your client.
  * (Recommended) If you only want DNS traffic to be passed to your Wirehole deployment, change the `Allowed IPs` to be `10.2.0.0/24`. Update your local DNS one the client to have `10.2.0.100` and the first DNS server and another (such as `1.1.1.1`) as your secondary.
  ```
[Interface]
Address = 10.6.0.2
PrivateKey = ABCDEFGHIJK1234567890ABCDEFGHIJK1234567890
ListenPort = 51820
DNS = 10.2.0.100

[Peer]
PublicKey = ABCDEFGHIJK1234567890ABCDEFGHIJK1234567890
Endpoint = WIREHOLEIP:51820
AllowedIPs = 10.2.0.0/24
  ```
  * If you want all traffic to flow through through the VPN (there is a 20TB monthly limit on default Hetzner Cloud 1cpu/2GB server, which is very generous, but sitll be aware) then you do not need to update the `Allowed IPs`, but you will still want to update your client DNS.
  ```
  [Interface]
Address = 10.6.0.2
PrivateKey = ABCDEFGHIJK1234567890ABCDEFGHIJK1234567890
ListenPort = 51820
DNS = 10.2.0.100

[Peer]
PublicKey = ABCDEFGHIJK1234567890ABCDEFGHIJK1234567890
Endpoint = WIREHOLEIP:51820
AllowedIPs = 0.0.0.0/0, ::/0
  ```
13. Start your Wireguard client and navigate to `http://10.2.0.100/admin` in your browser to reach the Pi-Hole admin panel. If you can reach it, everything should be working as expected.