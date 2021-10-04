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

For Your Information:

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

.. code-block:: yaml
    extra_admin_username: cyber
    extra_app_vm_size: Standard_B2ms
    extra_hub_name: MyHub
    extra_jumphost:
      acl_src_ips:
        - 10.0.0.0/8
      name: jumphost
      vm_size: Standard_DS1_v2
    extra_k8s_version: 1.21.2
    extra_location: eastus2
    extra_owner_email: me@acme.com
    extra_platform_tags: environment=labSE project=CloudBuilder
    extra_ssh_crt: ...
    extra_ssh_key: ...
    extra_volterra_site_id: 1
    extra_zone_name: cni-nodesandpods


Infra Ingress Controller
=========================================
Deploy an Ingress Controller.
Launch the workflow template ``wf-k8s-infra-create-ingress-controller`` and fulfill the survey.

For Your Information:

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
``poc-k8s-deploy_appolicy_generic``                             Create a K8S App Protect Policy                     ``playbooks/poc-k8s.yaml``                      ``deploy_appolicy_generic``                     localhost
``poc-letsencrypt-get_certificate``                             Create a CSR and a Let's Encrypt challenge          ``playbooks/poc-letsencrypt.yaml``              ``get_certificate``                             localhost                                       f5-cloudbuilder-mgmt
``poc-k8s-deploy_kibana``                                       Create Kibana ingress and service                   ``playbooks/poc-k8s.yaml``                      ``deploy_kibana``                               localhost
``poc-k8s-deploy_gslb_virtual_server``                          Create a LB record 'Kibana ingress 'in F5 CS        ``playbooks/poc-k8s.yaml``                      ``deploy_gslb_virtual_server``                  localhost
``poc-letsencrypt-assert_crt``                                  Check that the CRT is still valid                   ``playbooks/poc-letsencrypt.yaml``              ``assert_crt``                                  localhost                                       f5-cloudbuilder-mgmt
``poc-k8s-deploy_acme_challenge_vsr``                           Create a VSR to get ACME challenge validation       ``playbooks/poc-k8s.yaml``                      ``deploy_acme_challenge_vsr``                   localhost
``poc-letsencrypt-get_certificate``                             Validate challenge by Let's Encrypt + get CRT       ``playbooks/poc-letsencrypt.yaml``              ``get_certificate``                             localhost                                       f5-cloudbuilder-mgmt
``poc-k8s-deploy_kibana``                                       Update Kibana ingress with valid CRT                ``playbooks/poc-k8s.yaml``                      ``deploy_kibana``                               localhost
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================

==============================================  =============================================
Extra variable                                  Description
==============================================  =============================================
extra_namespace                                 External (ELB) or Internal (ILB)
extra_wildcard_tls_crt                          CRT used when no NGINX Sever block match
extra_wildcard_tls_key                          KEY used when no NGINX Sever block match
==============================================  =============================================


.. code-block:: yaml

    extra_app:
      domain: f5app.dev
      gslb_location:
        - eu
      name: kibana
    extra_jumphost:
      name: jumphost
    extra_namespace: external-ingress-controller
    extra_nginx_ic_version: 1.12.1
    extra_ns_prefix: infra
    extra_project: f5-k8s-ctfd
    extra_volterra_site_id: 1
    extra_wildcard_tls_crt: ...
    extra_wildcard_tls_key: ...

Lab 1 - Ingress
=========================================

Deploy application Arcadia using an ``ingress`` manifest.
Launch the workflow template ``wf-k8s-lab1-publish-app_arcadia`` and fulfill the survey.

For Your Information:

=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
Job template                                                    objective                                           playbook                                        activity or play targeted in role               inventory                                       credential
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
``poc-aks-get-registry_info``                                   Get ACR login server, username and password         ``playbooks/poc-aks.yaml``                      ``get-registry_info``                           CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-azure-get-vm-jumphost``                                   Get Public IP of Jumphost                           ``playbooks/poc-azure.yaml``                    ``get-vm-jumphost``                             CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-k8s-create_app_image``                                    Create an image of given App                        ``playbooks/poc-k8s_jumphost.yaml``             ``create_app_image``                            localhost                                       f5-k8s-ctfd-jumphost
``poc-aks-get-cluster_info``                                    Get kubeconfig                                      ``playbooks/poc-aks.yaml``                      ``get-cluster_info``                            CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-letsencrypt-get_certificate``                             Create a CSR and a Let's Encrypt challenge          ``playbooks/poc-letsencrypt.yaml``              ``get_certificate``                             localhost                                       f5-cloudbuilder-mgmt
``poc-k8s-deploy_app_arcadia``                                  Create Arcadia ingress and service                  ``playbooks/poc-k8s.yaml``                      ``deploy_app_arcadia``                          localhost
``poc-k8s-deploy_gslb_ingress``                                 Create a LB record 'Arcadia ingress 'in F5 CS       ``playbooks/poc-k8s.yaml``                      ``deploy_gslb_ingress``                         localhost
``poc-letsencrypt-assert_crt``                                  Check that the CRT is still valid                   ``playbooks/poc-letsencrypt.yaml``              ``assert_crt``                                  localhost                                       f5-cloudbuilder-mgmt
``poc-k8s-deploy_acme_challenge_master``                        Create a VSR to get ACME challenge validation       ``playbooks/poc-k8s.yaml``                      ``deploy_acme_challenge_master``                localhost
``poc-letsencrypt-get_certificate``                             Validate challenge by Let's Encrypt + get CRT       ``playbooks/poc-letsencrypt.yaml``              ``get_certificate``                             localhost                                       f5-cloudbuilder-mgmt
``poc-k8s-deploy_app_arcadia``                                  Update Arcadia ingress with valid CRT               ``playbooks/poc-k8s.yaml``                      ``deploy_app_arcadia``                          localhost
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================

==============================================  =======================================================================================================
Extra variable                                  Description
==============================================  =======================================================================================================
extra_waf_policy_level                          Policy fetch from `SecOps repo <https://github.com/nergalex/f5-nap-policies/tree/master/policy/core>`_
==============================================  =======================================================================================================


.. code-block:: yaml

    extra_ns_prefix: lab1
    extra_project: f5-k8s-ctfd
    extra_app_swagger_url: none
    extra_jumphost:
      name: jumphost
    extra_app:
      name: arcadia
      domain: f5app.dev
      gslb_location:
        - eu
      components:
        - name: main
          location: /
          source_image: 'https://gitlab.com/arcadia-application/main-app.git'
        - name: app2
          location: /api
          source_image: 'https://gitlab.com/arcadia-application/app2.git'
        - name: app3
          location: /app3
          source_image: 'https://gitlab.com/arcadia-application/app3.git'
        - name: backend
          location: /files
          source_image: 'https://gitlab.com/arcadia-application/back-end.git'

Lab 3 - WAF
=========================================

Deploy application Cafe using a ``VirtualServer`` manifest.
Launch the workflow template ``wf-k8s-lab3-publish-app_cafe`` and fulfill the survey.

For Your Information:

=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
Job template                                                    objective                                           playbook                                        activity or play targeted in role               inventory                                       credential
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
``poc-aks-get-cluster_info``                                    Get kubeconfig                                      ``playbooks/poc-aks.yaml``                      ``get-cluster_info``                            CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-letsencrypt-get_certificate``                             Create a CSR and a Let's Encrypt challenge          ``playbooks/poc-letsencrypt.yaml``              ``get_certificate``                             localhost                                       f5-cloudbuilder-mgmt
``poc-k8s-deploy_app_cafe``                                     Create Cafe ingress and service                     ``playbooks/poc-k8s.yaml``                      ``deploy_app_cafe``                             localhost
``poc-k8s-deploy_gslb_virtual_server``                          Create a LB record 'Kibana ingress 'in F5 CS        ``playbooks/poc-k8s.yaml``                      ``deploy_gslb_virtual_server``                  localhost
``poc-letsencrypt-assert_crt``                                  Check that the CRT is still valid                   ``playbooks/poc-letsencrypt.yaml``              ``assert_crt``                                  localhost                                       f5-cloudbuilder-mgmt
``poc-k8s-deploy_acme_challenge_vsr``                           Create a VSR to get ACME challenge validation       ``playbooks/poc-k8s.yaml``                      ``deploy_acme_challenge_vsr``                   localhost
``poc-letsencrypt-get_certificate``                             Validate challenge by Let's Encrypt + get CRT       ``playbooks/poc-letsencrypt.yaml``              ``get_certificate``                             localhost                                       f5-cloudbuilder-mgmt
``poc-k8s-deploy_app_cafe``                                     Update Cafe ingress with valid CRT                  ``playbooks/poc-k8s.yaml``                      ``deploy_app_cafe``                             localhost
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================

==============================================  =======================================================================================================
Extra variable                                  Description
==============================================  =======================================================================================================
extra_waf_policy_level                          Policy fetch from `SecOps repo <https://github.com/nergalex/f5-nap-policies/tree/master/policy/core>`_
==============================================  =======================================================================================================


.. code-block:: yaml

    extra_app:
      components:
        - name: coffee
          location: /coffee
          image: nginxdemos/nginx-hello
        - name: tea
          location: /tea
          image: 'nginxdemos/nginx-hello:plain-text'
      domain: f5app.dev
      gslb_location:
        - eu
      name: cafe
    extra_app_swagger_url: none
    extra_jumphost:
      name: jumphost
    extra_ns_prefix: lab3
    extra_project: f5-k8s-ctfd
    extra_volterra_site_id: 1
    extra_waf_policy_level: low
