# Apex-Colorado-3.0
=========Install APEX Colorado==========
1. Install VMware for Centos 7
2. VM hardware require: 40Gb RAM and 300Gb HDD
	2 network card: 1 card NAT and 1 card Host-only
3. Download Apex package 
http://artifacts.opnfv.org/apex.html
   - opnfv-apex-common-colorado-3.0.noarch.rpm
   - opnfv-apex-3.0-colorado-3.0.noarch.rpm  (Full)
   - opnfv-apex-undercloud-3.0-colorado-3.0.noarch.rpm 
   - opnfv-apex-opendaylight-sfc-3.0-20161019.noarch.rpm (Only opendaylight)
e
   - python34-markupsafe-0.23-9.el7.centos.x86_64.rpm
   - python3-jinja2-2.8-5.el7.centos.noarch.rpm
   - python3-ipmi-0.3.0-1.noarch.rpm
   
4. Setup KVM and Libvirt
- Check CPU support virtualization (must be #0) or not (=0).
grep -E '(vmx|svm)' /proc/cpuinfo
egrep -c '(vmx|svm)' /proc/cpuinfo 

5. Install KVM and its associated packages
- Ref: http://www.linuxtechi.com/install-kvm-hypervisor-on-centos-7-and-rhel-7/
- Install KVM and its associated packages
sudo yum install qemu-kvm qemu-img virt-manager libvirt libvirt-python libvirt-client virt-install virt-viewer bridge-utils
- Start and enable the libvirtd service
sudo systemctl start libvirtd
sudo systemctl enable libvirtd
- Run the beneath command to check whether KVM module is loaded or not
lsmod | grep kvm
- In Case you have Minimal CentOS 7 and RHEL 7 installation , then virt-manger will not start for that you need to install x-window package.
sudo  yum install "@X Window System" xorg-x11-xauth xorg-x11-fonts-* xorg-x11-utils -y
- Reboot the Server and then try to start virt manager.
sudo shutdown -r now

6. Install openstack Newton (OPNVF Colorado use Openstack Colorado)
sudo yum install -y https://www.rdoproject.org/repos/rdo-release.rpm

sudo yum install -y centos-release-openstack-newton

sudo yum install epel-release

sudo yum install python34

8. Install downloaded packe
sudo yum -y install python34-markupsafe-0.23-9.el7.centos.x86_64.rpm python3-jinja2-2.8-5.el7.centos.noarch.rpm python3-vmx-0.3.0-1.noarch.rpm

sudo yum -y install opnfv-apex-3.0-colorado-3.0.noarch.rpm opnfv-apex-common-colorado-3.0.noarch.rpm opnfv-apex-undercloud-3.0-colorado-3.0.noarch.rpm 

9. Run opnfv-deploy
sudo opnfv-deploy -v -d /etc/opnfv-apex/os-odl_l2-sfc-noha.yaml -n /etc/opnfv-apex/network_settings.yaml

10. If opnfv-deploy error, run "opnfv-clean" to clean configed
sudo opnfv-clean

===============================Tim Rozet==================================
1. Deploy an OPNFV Apex deployment with the SFC scenario:
    a.  Requires a CentOS 7 box with at least 16 GB of RAM, and 40 GB Disk Space, with ROOT access
    b.  Download the latest Apex RPMs from artifacts.opnfv.org:
           - opnfv-apex-opendaylight-sfc-3.0-{date}.noarch.rpm
           - opnfv-apex-common-3.0-{date}.noarch.rpm
           - opnfv-apex-undercloud-3.0-{date}.noarch.rpm
    c.  Install RPMs.  Note, you should provide all 3 RPMs as arguments to yum: yum -y install <rpm1> <rpm2> <rpm3>
    d.  opnfv-deploy -v -d /etc/opnfv-apex/os-odl_l2-sfc-noha.yaml -n /etc/opnfv-apex/network_settings.yaml
    e.  This will take approximately 20-30 minutes depending on your system.

2. Install/Configure/Start Tacker:
    a.  git clone https://github.com/trozet/sfc-random.git 
    b.  ./sfc-random/tacker_config.sh
		Edit "tacker_config.sh"
		"https://github.com/trozet/tacker.git -b SFC_colorado"
    c.  At the end of the script, it will tell you to ssh as heat-admin to the control node.
		
	Notice: /Stage[main]/Tacker::Service/Service[tacker]/ensure: ensure changed 'stopped' to 'running'
	"overcloudrc"
	"sfcrc"
	Tacker running on Controller: 192.0.2.5.  Please ssh heat-admin@192.0.2.5 to access
	"sudo ssh heat-admin@192.0.2.5"	
	
    d.  After connecting to the control node, source sfcrc.

*Note: The following steps are optional and are only for demo'ing SFC:

3. Create the "net_mgmt" (SFC) network:
    a.  neutron net-create net_mgmt --provider:network_type=vxlan --provider:segmentation_id 1005
    b.  neutron subnet-create net_mgmt 123.123.123.0/24

4. Create Glance SFC Image and custom Flavor
    a.  wget https://www.dropbox.com/s/focu44sh52li7fz/sfc_cloud.qcow2
    b.  openstack image create sfc --public --file ./sfc_cloud.qcow2 --disk-format qcow2 --container-format bare
    c.  openstack flavor create custom --ram 1000 --disk 5 --public

5. Import VNFD for test-VNF:
    a.  git clone https://github.com/trozet/sfc-random
    b.  tacker vnfd-create --vnfd-file ./sfc-random/test-vnfd.yaml --name test-vnfd
	Check list of VNF Description
	"tacker vnfd-list"
	
6. Deploy VNFs:
    a.  tacker vnf-create --name testVNF1 --vnfd-name test-vnfd (note it may take 5-10 min to boot)
		Check location of VNF
		"heat resource-list be6695cf-29db-44a5-8258-08d5ae93b715"
    b.  Tacker will kick off creating the VNFs via Heat.  
		For some retackason it does this as the service tenant.  
		So you can check heat by using the "heat" user and doing "heat stack-list".
		Check "tacker vnf-list"
    c.  Wait a few minutes and then check VNF status is ACTIVE: tacker vnf-list
		If want to delete a VM, you will delete in 2 case:
		- "nova delete id_instance"
		- "tacker vnf-delete id_instance"
    d.  Login to horizon by entering the VM IP into your web browser as "tacker" password "tacker"
		http://192.168.37.10/dashboard 
		
    e.  Go to Compute->Instances.  Click on the tacker VNF instance.
		"sudo ip netns" ==> id_netns
		"sudo ip netns exec qdhcp-id_netns ssh root@ip_instance"
		
    f.  Click Console and login to the VNF with user "root", password "octopus"
	g.  in the console: "service iptables stop"
    g.  type "python /root/vxlan_tool.py -i eth0 -d forward -v on"
	h.  If the VNF do not obitain IP add ==> Restart Opendaylight
		"sudo service opendaylight restart"
		
7. Create HTTP Client and Server:
    a.  Using either Horizon or CLI create a cirros instance named http_client, with default flavor (m1.tiny)
    b.  For creating HTTP server you can create another instance using the sfc image (custom flavor), 1000MB
        RAM and 5GB disk.  The image also supports cloud-init, so you can insert your SSH key as well.
    c.  Login to the http_server, and disable iptables: "service iptables stop"
    d.  Start simple python http server: "python -m SimpleHTTPServer 80"

8. Create SFC via python client(will render the chain into ODL):
    a.  tacker sfc-create --name mychain --chain testVNF1
    b.  This will create a chain in ODL, with 1 Rendered Service Path
    c.  tacker sfc-show mychain (check to make sure your SFC is now "Active")
    d.  You can now see NSH flows added to OVS br-int by doing the following:
        sudo ovs-ofctl -O OpenFlow13 dump-flows br-int

9. Create SFC Classifier with netvirt
    a.  tacker sfc-classifier-create --name myclass --chain mychain --match source_port=2000,dest_port=80,protocol=6
    b.  This will create a classifier for matching HTTP traffic to go onto your chain
    c.  Check OVS to see if you have hte classifier flow:
        sudo ovs-ofctl dump-flows br-int -O openflow13 | grep tp_dst=80
        tcp,reg0=0x1,tp_dst=80 actions=move:NXM_NX_TUN_ID[0..31]->NXM_NX_NSH_C2[],set_nshc1:0,set_nsp:0x2d,set_nsi:255,load:0xc0a80104->NXM_NX_TUN_IPV4_DST[],load:0x2d->NXM_NX_TUN_ID[0..31],output:6

10. Ensure VXLAN workaround is configured on compute node
    a.  In another terminal, virsh domiflist instack
    b.  ssh stack@$(arp -a | grep $(virsh domiflist undercloud | grep default | awk '{print $5}') | grep -Eo "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+")
    c.  You are now connected to the undercloud VM (installer VM).  Do "source stackrc"
    d.  nova list | grep compute
    e.  ssh heat-admin@<compute ip>
    f.  sudo ifconfig br-int up
    g.  sudo ip route add 123.123.123.0/24 dev br-int

11. Test SFC!
    a.  Go to horizon, click Compute->Instances, note the IP of the http server/http client
    b.  Now click the VNF and go to its console
    c.  On the VM terminal "ip netns list" to find the ID of the qdhcp namespace
    d.  ip netns exec <qdhcp ns ID> ssh cirros@<http client ip>; password is cubswin:)
    e.  Now in cirros, curl --local-port 2000 <http server ip>
    f.  Verify you see packets being redirected through the SFC (hitting the VNF in the horizon console)
    g.  Verify you receive 200 OK response from the http server

To re-stack (execute on host server):
1.  opnfv-clean (Note: this will destroy undercloud and overcloud)
2. Execute step 1d.


==========================================
=======stackrc=========
export NOVA_VERSION=1.1
export OS_PASSWORD=$(sudo hiera admin_password)
export OS_AUTH_URL=http://192.0.2.1:5000/v2.0
export OS_USERNAME=admin
export OS_TENANT_NAME=admin
export COMPUTE_API_VERSION=1.1
export OS_NO_CACHE=True
export OS_CLOUDNAME=undercloud
export OS_IMAGE_API_VERSION=1

=======overcloudrc=======
export OS_NO_CACHE=True
export OS_CLOUDNAME=overcloud
export OS_AUTH_URL=http://192.168.37.10:5000/v2.0
export NOVA_VERSION=1.1
export COMPUTE_API_VERSION=1.1
export OS_USERNAME=admin
export no_proxy=,192.168.37.10,192.0.2.3
export OS_PASSWORD=ZEcD2hCsCsTKXvJaQkeTNBtuw
export PYTHONWARNINGS="ignore:Certificate has no, ignore:A true SSLContext object is not available"
export OS_TENANT_NAME=admin
export SDN_CONTROLLER_IP=192.0.2.5
=========================================
