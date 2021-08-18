NGINX Ingress Controller workshop - admin guide
##############################################################
Thank you for your interest in *NGINX Ingress Controller workshop*.
*NGINX Ingress Controller workshop* is a F5 community lab that helps DevSecOps teams manage NGINX Ingress Controller deployments by adding control, visibility and security to their Apps.

If you are a student, please visit `User guide <https://f5-k8s-ctfd.docs.emea.f5se.com/>`_.

The *NGINX Ingress Controller workshop* Administration Guide documents the administration of the lab platform and each lab through `Red Hat Ansible Tower <https://www.ansible.com/products/tower>`_.

.. contents:: Contents
    :local:

Pre-requisites
*****************************************
Access to Tower
=========================================
- Allow your Public IP address to access Tower: Network security group *nsg-tower* >> rule *https*
- Connect to `Tower <https://tower-cloudbuilderf5.eastus2.cloudapp.azure.com>`_ using provided credentials

Deployment
*****************************************

Common platform
=========================================
Deploy a platform per student.
Launch the workflow template ``wf-aks-create-infra`` and fulfill the survey.

For information, the workflow does:

=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
Job template                                                    objective                                           playbook                                        activity or play targeted in role               inventory                                       credential
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
``poc-azure-create-spoke-aks``                                  Create a resource group and network objects         ``playbooks/poc-azure.yaml``                    ``create-spoke-aks``                            CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-aks-create-registry``                                     Create an Azure Container Registry                  ``playbooks/poc-aks.yaml``                      ``create-registry``                             CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-aks-create-cluster``                                      Create an Azure Kubernetes Service                  ``playbooks/poc-aks.yaml``                      ``create-cluster``                              CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-azure-create-vm-jumphost``                                Create an Linux host as Jumphost                    ``playbooks/poc-azure.yaml``                    ``create-vm-jumphost``                          CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-aks-get-cluster_info``                                    Get kubeconfig                                      ``playbooks/poc-aks.yaml``                      ``get-cluster_info``                            CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-azure-get-vm-jumphost``                                   Get Public IP of Jumphost                           ``playbooks/poc-azure.yaml``                    ``get-vm-jumphost``                             CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-k8s-copy_kubeconfig``                                     Copy kubeconfig on Jumphost                         ``playbooks/poc-k8s_jumphost.yaml``             ``copy_kubeconfig``                             localhost                                       f5-k8s-ctfd-jumphost
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================

==============================================  =============================================
Extra variable                                  Description
==============================================  =============================================
``extra_admin_username``                        admin username on Jumphost
``extra_app_vm_size``                           VM type in K8S's VMSS
``extra_hub_name``                              HUB platform connected to all SPOKES
``extra_jumphost.vm_size``                      VM type for jumphost
``extra_k8s_version``                           AKS version
``extra_location``                              Azure region
``extra_owner_email``                           e-mail address of student
``extra_volterra_site_id``                      site ID of student, aka student ID
==============================================  =============================================

Infra Ingress Controller
=========================================
Deploy an Ingress Controller.
Launch the workflow template ``wf-k8s-infra-create-ingress-controller`` and fulfill the survey.

For information, the workflow does:

=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
Job template                                                    objective                                           playbook                                        activity or play targeted in role               inventory                                       credential
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
``poc-aks-get-registry_info``                                   Get ACR login server, username and password         ``playbooks/poc-aks.yaml``                      ``get-registry_info``                           CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-azure-get-vm-jumphost``                                   Get Public IP of Jumphost                           ``playbooks/poc-azure.yaml``                    ``get-vm-jumphost``                             CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-aks-get-cluster_info``                                    Get kubeconfig                                      ``playbooks/poc-aks.yaml``                      ``get-cluster_info``                            CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-k8s-create_nap_converter_image``                          Create an image of NAP converter tool               ``playbooks/poc-k8s_jumphost.yaml``             ``create_nap_converter_image``                  localhost                                       f5-k8s-ctfd-jumphost
``poc-k8s-create_nginx_ic_image``                               Create an image of IC with App Protect              ``playbooks/poc-k8s_jumphost.yaml``             ``create_nginx_ic_image``                       localhost                                       f5-k8s-ctfd-jumphost
``poc-k8s-deploy_nginx_ic``                                     Create a K8S deployment of IC                       ``playbooks/poc-k8s.yaml``                      ``deploy_nginx_ic``                             localhost
``poc-k8s-create_nap_log_format``                               Create a K8S App Protect >> log format              ``playbooks/poc-k8s.yaml``                      ``create_nap_log_format``                       localhost

ToDo

=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================

==============================================  =============================================
Extra variable                                  Description
==============================================  =============================================
ToDo
==============================================  =============================================
