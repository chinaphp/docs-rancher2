---
title: 无法获取 Job 完成状态
---

> #### **重要：RKE add-on 安装 仅支持至 Rancher v2.0.8 版本**
>
> 你可以使用 Helm chart 在 Kubernetes 集群中安装 Rancher。获取更多细节，请参考[安装 Kubernetes - 安装概述](/docs/installation/k8s-install/_index)。
>
> 如果您正在使用 RKE add-on 安装的方法，请参阅[从一个 RKE Add-on 安装的 Rancher 迁移到 Helm 安装](/docs/upgrades/upgrades/migrating-from-rke-add-on/_index) 获取如何迁移至使用 helm chart 的更多细节。

要调试有关此错误的问题，您需要下载命令行工具 `kubectl`。请参阅[安装和设置 kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)，查看如何在您的平台上下载 `kubectl`。

当您对`rancher-cluster.yml`进行更改后，您将必须运行`rke remove --config rancher-cluster.yml`来清理节点，避免与以前的错误配置冲突。

## Failed to deploy addon execute job [rke-user-includes-addons]: Failed to get job complete status

插件定义执行过程中有问题，您可以运行以下命令来获取日志查看作业失败的原因：

```
kubectl --kubeconfig kube_config_rancher-cluster.yml logs -l job-name=rke-user-addon-deploy-job -n kube-system
```

## error: error converting YAML to JSON: yaml: line 9

`rancher-cluster.yml` 中插件定义的结构有错误。在 addons 部分中指定的资源中，YAML 的结构存在错误。根据 `yaml line 9` 所标识的引用行号寻找错误原因。

**需要检查**

- 是否将每个 base64 编码的证书字符串直接放置在密钥之后，例如: `tls.crt: LS01...`，在此之前，之间或之后不应有换行符/空格。

- YAML 的格式是否正确，每个缩进应为 2 个空格如模板文件中所示。

- 运行以下命令验证证书的完整性，如果存在任何错误, 命令输出将显示。
  - Linux：`cat MyCertificate | base64 -d`
  - Mac OS：`cat MyCertificate | base64 -D`

## Error from server (BadRequest): error when creating “/etc/config/rke-user-addon.yaml”: Secret in version “v1” cannot be handled as a Secret

某个证书的 base64 字符串错误。日志消息将尝试向您显示字符串的哪一部分未被识别为有效的 base64。

**需要检查**

通过运行以下命令之一检查 base64 字符串是否有效：

```

## MacOS

echo BASE64_CRT | base64 -D

## Linux

echo BASE64_CRT | base64 -d

## Windows

certutil -decode FILENAME.base64 FILENAME.verify
```

## The Ingress “cattle-ingress-http” is invalid: spec.rules[0].host: Invalid value: “IP”: must be a DNS name, not an IP address

主机值只能包含一个主机名, 因为 ingress controller 需要该主机名来匹配主机名并传递给正确的后端。
