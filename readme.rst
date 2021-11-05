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


Lab 1-4 - Edge Proxy - Ingress Controller
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

Lab 1 - App arcadia using ingress resource
==========================================

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

Lab 2 - App cafe using VS(R) resource
=========================================

Deploy application Cafe using a ``VirtualServer`` manifest.
Launch the workflow template ``wf-k8s-lab2-publish-app_cafe`` and fulfill the survey.

For Your Information:

=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
Job template                                                    objective                                           playbook                                        activity or play targeted in role               inventory                                       credential
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
``poc-aks-get-cluster_info``                                    Get kubeconfig                                      ``playbooks/poc-aks.yaml``                      ``get-cluster_info``                            CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-letsencrypt-get_certificate``                             Create a CSR and a Let's Encrypt challenge          ``playbooks/poc-letsencrypt.yaml``              ``get_certificate``                             localhost                                       f5-cloudbuilder-mgmt
``poc-k8s-deploy_app_lab2-cafe``                                Create Cafe VS(R)s                                  ``playbooks/poc-k8s.yaml``                      ``deploy_app_lab2-cafe``                        localhost
``poc-k8s-deploy_gslb_virtual_server``                          Create a LB record 'Kibana ingress 'in F5 CS        ``playbooks/poc-k8s.yaml``                      ``deploy_gslb_virtual_server``                  localhost
``poc-letsencrypt-assert_crt``                                  Check that the CRT is still valid                   ``playbooks/poc-letsencrypt.yaml``              ``assert_crt``                                  localhost                                       f5-cloudbuilder-mgmt
``poc-k8s-deploy_acme_challenge_vsr``                           Create a VSR to get ACME challenge validation       ``playbooks/poc-k8s.yaml``                      ``deploy_acme_challenge_vsr``                   localhost
``poc-letsencrypt-get_certificate``                             Validate challenge by Let's Encrypt + get CRT       ``playbooks/poc-letsencrypt.yaml``              ``get_certificate``                             localhost                                       f5-cloudbuilder-mgmt
``poc-k8s-deploy_app_lab2-cafe``                                Update Edge Proxy with valid CRT                    ``playbooks/poc-k8s.yaml``                      ``deploy_app_lab2-cafe``                             localhost
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================

==============================================  =======================================================================================================
Extra variable                                  Description
==============================================  =======================================================================================================
extra_waf_policy_level                          Policy fetch from `SecOps repo <https://github.com/nergalex/f5-nap-policies/tree/master/policy/core>`_
==============================================  =======================================================================================================


.. code-block:: yaml

    extra_ns_prefix: lab2
    extra_project: f5-k8s-ctfd
    extra_jumphost:
      name: jumphost
    extra_app:
      name: cafeapp
      domain: f5app.dev
      gslb_location:
        - eu
      components:
        - name: coffee-v1
          location: /coffee
          image: 'nginxdemos/nginx-hello:plain-text'
        - name: tea-v1
          location: /tea
          image: 'nginxdemos/nginx-hello:plain-text'
        - name: coffee-v2
          location: /coffee
          image: 'nginxdemos/nginx-hello'

Lab 3 - App cafe + WAF protection
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

Lab 4 - Micro-Proxy -  Ingress Controller
=========================================
Deploy an Ingress Controller dedicated for an Application and accessible only from inside the cluster.
Launch the workflow template ``wf-k8s-lab4-create-ingress-controller`` and fulfill the survey.

For Your Information:

=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
Job template                                                    objective                                           playbook                                        activity or play targeted in role               inventory                                       credential
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
``poc-aks-get-registry_info``                                   Get ACR login server, username and password         ``playbooks/poc-aks.yaml``                      ``get-registry_info``                           CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-azure-get-vm-jumphost``                                   Get Public IP of Jumphost                           ``playbooks/poc-azure.yaml``                    ``get-vm-jumphost``                             CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-aks-get-cluster_info``                                    Get kubeconfig                                      ``playbooks/poc-aks.yaml``                      ``get-cluster_info``                            CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-k8s-create_nginx_ic_image``                               Create an image of IC with App Protect              ``playbooks/poc-k8s_jumphost.yaml``             ``create_nginx_ic_image``                       localhost                                       f5-k8s-ctfd-jumphost
``poc-k8s-deploy_nginx_ic``                                     Create a K8S deployment of IC                       ``playbooks/poc-k8s.yaml``                      ``deploy_nginx_ic``                             localhost
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================

.. code-block:: yaml

    extra_jumphost:
      name: jumphost
    extra_namespace: sentence-api
    extra_nginx_ic_version: 2.0.2
    extra_ns_prefix: lab4
    extra_volterra_site_id: 1
    extra_wildcard_tls_crt: ...
    extra_wildcard_tls_key: ...

Lab 5 - Edge Proxy - Managed NAP
=========================================

Deploy NGINX App Protect containerized instances managed by NGINX Controller.
Launch the workflow template ``wf-k8s-lab5-create-waap-managed`` and fulfill the survey.

For Your Information:

=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
Job template                                                    objective                                           playbook                                        activity or play targeted in role               inventory                                       credential
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
``poc-nginx_controller-lab_k8s_create_tenant``                  Create environment and RBAC in Controller           ``playbooks/poc-nginx_controller.yaml``         ``lab_k8s_create_tenant``                       localhost
``poc-nginx_controller-lab_k8s_get_license``                    Get NGINX+ license from Controller                  ``playbooks/poc-nginx_controller.yaml``         ``lab_k8s_get_license``                         localhost
``poc-aks-get-registry_info``                                   Get ACR login server, username and password         ``playbooks/poc-aks.yaml``                      ``get-registry_info``                           CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-azure-get-vm-jumphost``                                   Get Public IP of Jumphost                           ``playbooks/poc-azure.yaml``                    ``get-vm-jumphost``                             CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-aks-get-cluster_info``                                    Get kubeconfig                                      ``playbooks/poc-aks.yaml``                      ``get-cluster_info``                            CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-k8s-create_nginx_managed_image``                          Create an image of NAP managed by Controller        ``playbooks/poc-k8s_jumphost.yaml``             ``create_nginx_managed_image``                  localhost                                       f5-k8s-ctfd-jumphost
``poc-k8s-deploy_nginx_managed``                                Deploy NAP managed by Controller                    ``playbooks/poc-k8s.yaml``                      ``deploy_nginx_managed``                        localhost
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================

==============================================  =======================================================================================================
Extra variable                                  Description
==============================================  =======================================================================================================
``extra_volterra_site_id``                      site ID of student, aka student ID
==============================================  =======================================================================================================

.. code-block:: yaml

    extra_ns_prefix: lab5
    extra_namespace: nap-managed
    extra_jumphost:
      name: jumphost
    extra_nginx_controller:
      ip: 10.0.0.12
      username: *********
      password: *********
    extra_policies:
      - name: bot_prevention
        url: https://raw.githubusercontent.com/nergalex/f5-nap-policies/master/policy/arcadia_bot_prevention.json
      - name: owasp_web
        url: https://raw.githubusercontent.com/nergalex/f5-nap-policies/master/policy/owasp_web_nginx.json
      - name: owasp_api
        url: https://raw.githubusercontent.com/nergalex/f5-nap-policies/master/policy/owasp_api_nginx.json


Lab 5 - Micro-Proxy - Ingress Controller
=========================================
Deploy an Ingress Controller dedicated for an Application and accessible only from inside the cluster.
Launch the workflow template ``wf-k8s-lab5-create-ingress-controller`` and fulfill the survey.

For Your Information:

=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
Job template                                                    objective                                           playbook                                        activity or play targeted in role               inventory                                       credential
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
``poc-aks-get-registry_info``                                   Get ACR login server, username and password         ``playbooks/poc-aks.yaml``                      ``get-registry_info``                           CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-azure-get-vm-jumphost``                                   Get Public IP of Jumphost                           ``playbooks/poc-azure.yaml``                    ``get-vm-jumphost``                             CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-aks-get-cluster_info``                                    Get kubeconfig                                      ``playbooks/poc-aks.yaml``                      ``get-cluster_info``                            CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-k8s-create_nginx_ic_image``                               Create an image of IC with App Protect              ``playbooks/poc-k8s_jumphost.yaml``             ``create_nginx_ic_image``                       localhost                                       f5-k8s-ctfd-jumphost
``poc-k8s-deploy_nginx_ic``                                     Create a K8S deployment of IC                       ``playbooks/poc-k8s.yaml``                      ``deploy_nginx_ic``                             localhost
``poc-k8s-create_nap_api_gw_log_format``                        Create a K8S App Protect >> log to stderr           ``playbooks/poc-k8s.yaml``                      ``create_nap_api_gw_log_format``                localhost
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================

==============================================  =======================================================================================================
Extra variable                                  Description
==============================================  =======================================================================================================
``extra_volterra_site_id``                      site ID of student, aka student ID
==============================================  =======================================================================================================

.. code-block:: yaml

    extra_ns_prefix: lab5
    extra_namespace: sentence-api-managed
    extra_nginx_ic_version: 2.0.3
    extra_jumphost:
      name: jumphost
    extra_wildcard_tls_crt: ...
    extra_wildcard_tls_key: ...

Lab 5 - App sentence-api-managed
=========================================

Deploy application sentence API.
Launch the workflow template ``wf-k8s-lab5-publish-app_sentence_api`` and fulfill the survey.

For Your Information:

=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
Job template                                                    objective                                           playbook                                        activity or play targeted in role               inventory                                       credential
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
``poc-aks-get-registry_info``                                   Get ACR login server, username and password         ``playbooks/poc-aks.yaml``                      ``get-registry_info``                           CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-azure-get-vm-jumphost``                                   Get Public IP of Jumphost                           ``playbooks/poc-azure.yaml``                    ``get-vm-jumphost``                             CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-aks-get-cluster_info``                                    Get kubeconfig                                      ``playbooks/poc-aks.yaml``                      ``get-cluster_info``                            CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-k8s-create_sentence_image``                               Create an images of App components                  ``playbooks/poc-k8s_jumphost.yaml``             ``create_sentence_image``                       localhost                                       f5-k8s-ctfd-jumphost
``poc-k8s-deploy_gslb_service``                                 Publish 'WAAP managed' service on F5 DNS LB         ``playbooks/poc-k8s.yaml``                      ``deploy_gslb_service``                         localhost
``poc-letsencrypt-get_certificate``                             Create a CSR and a Let's Encrypt challenge          ``playbooks/poc-letsencrypt.yaml``              ``get_certificate``                             localhost                                       f5-cloudbuilder-mgmt
``poc-k8s-deploy_app_sentence-api-managed``                     Deploy sentence-api VS(R)s on Micro-Proxy IC        ``playbooks/poc-k8s.yaml``                      ``deploy_app_sentence-api-managed``             localhost
``poc-nginx_controller-lab_k8s_create_app_api``                 Deploy sentence-api on Edge-Proxy                   ``playbooks/poc-nginx_controller.yaml``         ``lab_k8s_create_app_api``                      localhost
``poc-letsencrypt-assert_crt``                                  Check that the CRT is still valid                   ``playbooks/poc-letsencrypt.yaml``              ``assert_crt``                                  localhost                                       f5-cloudbuilder-mgmt
``poc-k8s-deploy_acme_challenge_vsr_api_gw``                    Create a VSR to get ACME challenge validation       ``playbooks/poc-k8s.yaml``                      ``deploy_acme_challenge_vsr``                   localhost
``poc-letsencrypt-get_certificate``                             Validate challenge by Let's Encrypt + get CRT       ``playbooks/poc-letsencrypt.yaml``              ``get_certificate``                             localhost                                       f5-cloudbuilder-mgmt
``poc-nginx_controller-lab_k8s_create_app_api``                 Update Edge-Proxy with valid CRT                    ``playbooks/poc-nginx_controller.yaml``         ``lab_k8s_create_app_api``                      localhost
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================

=====================================================  =======================================================================================================
Extra variable                                         Description
=====================================================  =======================================================================================================
``extra_volterra_site_id``                             site ID of student, aka student ID
``extra_app.components.XXX.nexthop_k8s_service.name``  Kubernetes local service name that will be resolved using local K8S DNS resolver service
=====================================================  =======================================================================================================

.. code-block:: yaml

    ---
    extra_ns_prefix: lab5
    extra_project: f5-k8s-ctfd
    extra_jumphost:
      name: jumphost
    extra_nginx_controller:
      ip: 10.0.0.12
      username: ***********
      password: ***********
    extra_app:
      name: sentence-api-managed
      domain: f5app.dev
      repo: https://github.com/fchmainy/nginx-aks-demo.git
      openapi_url: https://raw.githubusercontent.com/nergalex/f5-nap-policies/master/policy/open-api-files/sentence-api.f5app.dev.yaml
      acme_ingress_host_namespace: sentence-api-managed
      oidc:
        clientSecret: ***********
        clientID: ***********
        issuer: https://dev-431905.okta.com/oauth2/aus2zvhijqcz1rlq84x7
        scope: "openid"
      gslb_location:
        - eu
      components:
        - name: adjectives
          location: /adjectives
          image: 'registry.gitlab.com/sentence-application/adjectives/volterra:latest'
          source_image: 'Docker/adjectives'
          monitoring: none
          waf_policy:
            name: owasp_api
          nexthop_k8s_service:
            name: apigw-microapigw
        - name: animals
          location: /animals
          image: 'registry.gitlab.com/sentence-application/animals/volterra:latest'
          source_image: 'Docker/animals'
          monitoring: none
          waf_policy:
            name: owasp_api
          nexthop_k8s_service:
            name: apigw-microapigw
        - name: colors
          location: /colors
          image: 'registry.gitlab.com/sentence-application/colors/volterra:latest'
          source_image: 'Docker/colors'
          monitoring: none
          waf_policy:
            name: owasp_api
          nexthop_k8s_service:
            name: apigw-microapigw
        - name: locations
          location: /locations
          image: 'registry.gitlab.com/sentence-application/locations/volterra:latest'
          source_image: 'Docker/locations'
          monitoring: none
          waf_policy:
            name: owasp_api
          nexthop_k8s_service:
            name: apigw-microapigw
        - name: generator
          location: /
          image: 'registry.gitlab.com/sentence-application/generator/volterra-v0:dynamic'
          source_image: 'Docker/generator-via-api-gw'
          monitoring: none
          env:
            - name: NAMESPACE
              value: lab5-sentence-api-managed
          waf_policy:
            name: owasp_api
          nexthop_k8s_service:
            name: apigw-microapigw

Lab 5 - App sentence-front-managed
=========================================

Deploy application sentence web frontend.
Launch the workflow template ``wf-k8s-lab5-publish-app_sentence_front`` and fulfill the survey.

For Your Information:

=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
Job template                                                    objective                                           playbook                                        activity or play targeted in role               inventory                                       credential
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================
``poc-aks-get-registry_info``                                   Get ACR login server, username and password         ``playbooks/poc-aks.yaml``                      ``get-registry_info``                           CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-azure-get-vm-jumphost``                                   Get Public IP of Jumphost                           ``playbooks/poc-azure.yaml``                    ``get-vm-jumphost``                             CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-aks-get-cluster_info``                                    Get kubeconfig                                      ``playbooks/poc-aks.yaml``                      ``get-cluster_info``                            CMP_inv_CloudBuilderf5                          <Service Principal>
``poc-k8s-create_sentence_image``                               Create an images of App components                  ``playbooks/poc-k8s_jumphost.yaml``             ``create_sentence_image``                       localhost                                       f5-k8s-ctfd-jumphost
``poc-k8s-deploy_app_sentence-front-managed``                   Deploy App sentence-front                           ``playbooks/poc-k8s.yaml``                      ``deploy_app_sentence-front-managed``           localhost
``poc-k8s-deploy_gslb_service``                                 Publish 'WAAP managed' service on F5 DNS LB         ``playbooks/poc-k8s.yaml``                      ``deploy_gslb_service``                         localhost
``poc-letsencrypt-get_certificate``                             Create a CSR and a Let's Encrypt challenge          ``playbooks/poc-letsencrypt.yaml``              ``get_certificate``                             localhost                                       f5-cloudbuilder-mgmt
``poc-nginx_controller-lab_k8s_create_app``                     Deploy sentence-front on Edge-Proxy                 ``playbooks/poc-nginx_controller.yaml``         ``lab_k8s_create_app``                         localhost
``poc-letsencrypt-assert_crt``                                  Check that the CRT is still valid                   ``playbooks/poc-letsencrypt.yaml``              ``assert_crt``                                  localhost                                       f5-cloudbuilder-mgmt
``poc-k8s-deploy_acme_challenge_vsr_api_gw``                    Create a VSR to get ACME challenge validation       ``playbooks/poc-k8s.yaml``                      ``deploy_acme_challenge_vsr_api_gw``                   localhost
``poc-letsencrypt-get_certificate``                             Validate challenge by Let's Encrypt + get CRT       ``playbooks/poc-letsencrypt.yaml``              ``get_certificate``                             localhost                                       f5-cloudbuilder-mgmt
``poc-k8s-deploy_app_sentence-front-managed``                   Update Edge Proxy with valid CRT                    ``playbooks/poc-k8s.yaml``                      ``deploy_app_sentence-front-managed``           localhost
=============================================================   =============================================       =============================================   =============================================   =============================================   =============================================

=====================================================  =======================================================================================================
Extra variable                                         Description
=====================================================  =======================================================================================================
``extra_volterra_site_id``                             site ID of student, aka student ID
=====================================================  =======================================================================================================

.. code-block:: yaml

    extra_ns_prefix: lab5
    extra_project: f5-k8s-ctfd
    extra_jumphost:
      name: jumphost
    extra_nginx_controller:
      ip: 10.0.0.12
      username: ***********
      password: ***********
    extra_app:
      name: sentence-front-managed
      domain: f5app.dev
      acme_ingress_host_namespace: sentence-api-managed
      repo: 'https://github.com/fchmainy/nginx-aks-demo.git'
      gslb_location:
        - eu
      components:
        - name: frontend
          location: /
          image: 'registry.gitlab.com/sentence-application/frontend/frontend-ns-8080:latest'
          source_image: 'Docker/frontend-namespace-via-apigw'
          monitoring: none
          env:
            - name: NAMESPACE
              value: lab5-sentence-api-managed
          waf_policy:
            name: owasp_web
          nexthop_k8s_service:
            name: frontend