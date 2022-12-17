---
layout: post
title: Citrix DaaS Cloud Connector Load Balancer
hide_description: true
sitemap: false
categories: [NetScaler, CloudConnector]
---
This script will configure a secure Citrix DaaS Cloud Connector Load Balancer

```
# You will not need to do this part if you already have an AlwaysUp service present on your NetScaler
# ***************************************************************************************************

# Add Servers for always up service
add server always_up 127.0.0.1

# Add Services
add service svc_always_up always_up HTTP 80 -gslb NONE -maxClient 0 -maxReq 0 -cip DISABLED -usip NO -useproxyport YES -sp OFF -cltTimeout 180 -svrTimeout 360 -CKA NO -TCPB NO -CMP NO

# Add Responder Policies
add responder action res_act_http_to_https redirect "\"https://\" + HTTP.REQ.HOSTNAME.HTTP_URL_SAFE + HTTP.REQ.URL.PATH_AND_QUERY.HTTP_URL_SAFE" -responseStatusCode 302
add responder policy res_pol_http_to_https HTTP.REQ.IS_VALID res_act_http_to_https

# Add Always Up Monitor
add lb monitor mon_always_up PING -LRTM DISABLED -destIP 127.0.0.1

# Bind Always Up Monitor to Service
bind service svc_always_up -monitorName mon_always_up

# ***************************************************************************************************

# Add Servers for Cloud Connectors
add server daas_cc_svr_01 **INSERT_DAAS_CC_SERVER_IP**
add server daas_cc_svr_02 **INSERT_DAAS_CC_SERVER_IP**

# Add Service Group
add serviceGroup svc_grp_daas_cc_443 SSL -maxClient 0 -maxReq 0 -cip DISABLED -usip NO -useproxyport YES -cltTimeout 180 -svrTimeout 360 -CKA NO -TCPB NO -CMP NO

# Add Load Balancers
add lb vserver vsvr_daas_cc_80 HTTP **INSERT_DAAS_CC_LOAD_BALANCER_IP** 80 -persistenceType NONE -cltTimeout 180
add lb vserver vsvr_daas_cc_443 SSL **INSERT_DAAS_CC_LOAD_BALANCER_IP** 443 -persistenceType COOKIEINSERT -persistenceBackup SOURCEIP -cltTimeout 180

# Bind HTTP Server to Always Up Service and put redirect on
bind lb vserver vsvr_daas_cc_80 svc_always_up
bind lb vserver vsvr_daas_cc_80 -policyName res_pol_http_to_https -priority 100 -gotoPriorityExpression END -type REQUEST

# Bind Load Balancers to Service Group
bind lb vserver vsvr_daas_cc_443 svc_grp_daas_cc_443

# Add Custom Monitors
add lb monitor mon_daas_cc_443 CITRIX-XD-DDC -LRTM DISABLED -secure YES

# Bind Service Group to Director Server
bind serviceGroup svc_grp_daas_cc_443 mon_daas_cc_443 443

# Bind Service Group to Monitor
bind serviceGroup svc_grp_xendesktop_controller_443 -monitorName mon_controller_443

# Set Service Group Backend SSL Profile
set ssl serviceGroup svc_grp_xendesktop_controller_443 -sslProfile ns_default_ssl_profile_backend

# Set Load Balancer Front End SSL Profile
set ssl vserver vsvr_xendesktop_controller_443 -sslProfile ns_default_ssl_profile_secure_frontend

# Bind Certificate to Load Balancer
bind ssl vserver vsvr_xendesktop_controller_443 -certkeyName $Certificate_name$
```