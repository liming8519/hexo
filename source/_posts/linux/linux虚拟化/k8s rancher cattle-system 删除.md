---
title:  k8s rancher cattle-system 删除
tags:
- 虚拟化
categories: 
- linux 
date: 2019-11-19 02:00:00
---
> gitlab安装
<!-- more -->

### k8s rancher cattle-system 删除

```
[root@managementa ~]# kubectl get namespace cattle-system -o json
{
    "apiVersion": "v1",
    "kind": "Namespace",
    "metadata": {
        "annotations": {
            "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"v1\",\"kind\":\"Namespace\",\"metadata\":{\"annotations\":{},\"name\":\"cattle-system\"}}\n"
        },
        "creationTimestamp": "2019-11-15T07:06:06Z",
        "deletionTimestamp": "2019-11-18T06:19:15Z",
        "name": "cattle-system",
        "resourceVersion": "1078875",
        "selfLink": "/api/v1/namespaces/cattle-system",
        "uid": "3e15ccdc-a665-45bf-99e3-a2cd9d7937f2"
    },
    "spec": {
        "finalizers": [
            "kubernetes"
        ]
    },
    "status": {
        "conditions": [
            {
                "lastTransitionTime": "2019-11-18T06:19:20Z",
                "message": "Discovery failed for some groups, 1 failing: unable to retrieve the complete list of server APIs: metrics.k8s.io/v1beta1: the server is currently unable to handle the request",
                "reason": "DiscoveryFailed",
                "status": "True",
                "type": "NamespaceDeletionDiscoveryFailure"
            },
            {
                "lastTransitionTime": "2019-11-18T06:19:21Z",
                "message": "All legacy kube types successfully parsed",
                "reason": "ParsedGroupVersions",
                "status": "False",
                "type": "NamespaceDeletionGroupVersionParsingFailure"
            },
            {
                "lastTransitionTime": "2019-11-18T06:19:21Z",
                "message": "All content successfully deleted",
                "reason": "ContentDeleted",
                "status": "False",
                "type": "NamespaceDeletionContentFailure"
            }
        ],
        "phase": "Terminating"
    }
}
[root@managementa ~]# kubectl get namespace cattle-system -o json>cattle-system.json
[root@managementa ~]# vi cattle-system.json 
            "kubernetes"
        "finalizers": [
    "spec": {
    "spec": {
{
    "apiVersion": "v1",
    "kind": "Namespace",
    "metadata": {
        "annotations": {
            "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"v1\"
,\"kind\":\"Namespace\",\"metadata\":{\"annotations\":{},\"name\":\"cattle-system\"}}\n
"
        },
        "creationTimestamp": "2019-11-15T07:06:06Z",
        "deletionTimestamp": "2019-11-18T06:19:15Z",
        "name": "cattle-system",
        "resourceVersion": "1078875",
        "selfLink": "/api/v1/namespaces/cattle-system",
        "uid": "3e15ccdc-a665-45bf-99e3-a2cd9d7937f2"
    },
    "spec": {
    },
    "status": {
        "conditions": [
            {
                "lastTransitionTime": "2019-11-18T06:19:20Z",
@                                                                                      
"cattle-system.json" 43L, 1728C written
[root@managementa ~]# curl -k -H "Content-Type: application/json" -X PUT --data-binary @cattle-system.json http://192.168.106.117:9090/api/v1/namespaces/cattle-system/finalize
{
  "kind": "Namespace",
  "apiVersion": "v1",
  "metadata": {
    "name": "cattle-system",
    "selfLink": "/api/v1/namespaces/cattle-system/finalize",
    "uid": "3e15ccdc-a665-45bf-99e3-a2cd9d7937f2",
    "resourceVersion": "1078875",
    "creationTimestamp": "2019-11-15T07:06:06Z",
    "deletionTimestamp": "2019-11-18T06:19:15Z",
    "annotations": {
      "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"v1\",\"kind\":\"Namespace\",\"metadata\":{\"annotations\":{},\"name\":\"cattle-system\"}}\n"
    }
  },
  "spec": {
    
  },
  "status": {
    "phase": "Terminating",
    "conditions": [
      {
        "type": "NamespaceDeletionDiscoveryFailure",
        "status": "True",
        "lastTransitionTime": "2019-11-18T06:19:20Z",
        "reason": "DiscoveryFailed",
        "message": "Discovery failed for some groups, 1 failing: unable to retrieve the complete list of server APIs: metrics.k8s.io/v1beta1: the server is currently unable to handle the request"
      },
      {
        "type": "NamespaceDeletionGroupVersionParsingFailure",
        "status": "False",
        "lastTransitionTime": "2019-11-18T06:19:21Z",
        "reason": "ParsedGroupVersions",
        "message": "All legacy kube types successfully parsed"
      },
      {
        "type": "NamespaceDeletionContentFailure",
        "status": "False",
        "lastTransitionTime": "2019-11-18T06:19:21Z",
        "reason": "ContentDeleted",
        "message": "All content successfully deleted"
      }
    ]
  }
}
```