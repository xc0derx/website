---
title: "Продакшн окружение"
description: Создание кластера Kubernetes производственного качества
weight: 30
no_list: true
---
<!-- overview -->

Кластер Kubernetes производственного качества требует планирования и подготовки.
Если ваш кластер Kubernetes будет выполнять критически важные рабочие нагрузки, он должен быть настроен на отказоустойчивость.
На этой странице описаны шаги, которые вы можете предпринять для настройки производственно готового кластера или для повышения существующего кластера для использования в производстве.
Если вы уже знакомы с настройкой производственного кластера и хотите перейти к ссылкам, пропустите к
[Что дальше](#what-s-next).

<!-- body -->

## Производственные аспекты

Обычно среда производственного кластера Kubernetes имеет больше требований, чем среда Kubernetes для личного обучения, разработки или тестирования. Производственная среда может требовать безопасного доступа многим пользователям, постоянной доступности и ресурсов для адаптации к изменяющимся потребностям.

Пока вы решаете, где будет располагаться ваша производственная среда Kubernetes (на локальных серверах или в облаке), и какой уровень управления вы хотите взять на себя или передать другим, учитывайте, как ваши требования к кластеру Kubernetes зависят от следующих вопросов:

- *Доступность*: Kubernetes на одной машине [среда обучения](/docs/setup/#learning-environment) - это единая точка отказа.
  Создание кластера с высокой доступностью означает следующее:
  - Отделение плоскости управления от рабочих узлов.
  - Репликация компонентов плоскости управления на несколько узлов.
  - Балансировка нагрузки трафика на кластере {{< glossary_tooltip term_id="kube-apiserver" text="API сервер" >}}.
  - Наличие достаточного количества рабочих узлов или возможность их быстрого включения при необходимости, в соответствии с изменяющейся нагрузкой.

- *Масштабирование*: Если вы ожидаете, что ваша производственная среда Kubernetes будет испытывать стабильную нагрузку, возможно, вы сможете настроить необходимую мощность и закончить на этом. Однако, если вы ожидаете роста нагрузки со временем или резких изменений из-за таких факторов, как сезонность или особые события, вам необходимо спланировать, как масштабировать систему для снижения давления от большего количества запросов к плоскости управления и рабочим узлам, или масштабировать вниз для сокращения неиспользуемых ресурсов.

- *Безопасность и управление доступом*: У вас есть полные привилегии администратора на вашем собственном
  учебном кластере Kubernetes. Однако общие кластеры с важными рабочими нагрузками и более чем одним или двумя пользователями требуют более тщательного подхода к вопросу, кто и что может получать доступ к ресурсам кластера. Вы можете использовать контроль доступа на основе ролей ([RBAC](/docs/reference/access-authn-authz/rbac/)) и другие механизмы безопасности, чтобы убедиться, что пользователи и рабочие нагрузки могут получить доступ к необходимым ресурсам, сохраняя при этом безопасность рабочих нагрузок и самого кластера.
  Вы можете установить лимиты ресурсов, к которым пользватели и рабочие нагрузки могут получить доступ
  Вы можете устанавливать ограничения на ресурсы, доступные для пользователей и рабочих нагрузок, управляя [политиками](/docs/concepts/policy/) и
  [ресурсами для контейнеров](/docs/concepts/configuration/manage-resources-containers/).
Прежде чем создавать собственную производственную среду Kubernetes, рассмотрите возможность передачи части или всей этой работы [провайдерам готовых облачных решений](/docs/setup/production-environment/turnkey-solutions/) или другим [Партнерам Kubernetes](/partners/).

Варианты включают:

- *Serverless*: Just run workloads on third-party equipment without managing
  a cluster at all. You will be charged for things like CPU usage, memory, and
  disk requests.
- *Managed control plane*: Let the provider manage the scale and availability
  of the cluster's control plane, as well as handle patches and upgrades.
- *Managed worker nodes*: Configure pools of nodes to meet your needs,
  then the provider makes sure those nodes are available and ready to implement
  upgrades when needed.
- *Integration*: There are providers that integrate Kubernetes with other
  services you may need, such as storage, container registries, authentication
  methods, and development tools.

Whether you build a production Kubernetes cluster yourself or work with
partners, review the following sections to evaluate your needs as they relate
to your cluster’s *control plane*, *worker nodes*, *user access*, and
*workload resources*.

## Production cluster setup

In a production-quality Kubernetes cluster, the control plane manages the
cluster from services that can be spread across multiple computers
in different ways. Each worker node, however, represents a single entity that
is configured to run Kubernetes pods.

### Production control plane

The simplest Kubernetes cluster has the entire control plane and worker node
services running on the same machine. You can grow that environment by adding
worker nodes, as reflected in the diagram illustrated in
[Kubernetes Components](/docs/concepts/overview/components/).
If the cluster is meant to be available for a short period of time, or can be
discarded if something goes seriously wrong, this might meet your needs.

If you need a more permanent, highly available cluster, however, you should
consider ways of extending the control plane. By design, one-machine control
plane services running on a single machine are not highly available.
If keeping the cluster up and running
and ensuring that it can be repaired if something goes wrong is important,
consider these steps:

- *Choose deployment tools*: You can deploy a control plane using tools such
  as kubeadm, kops, and kubespray. See
  [Installing Kubernetes with deployment tools](/docs/setup/production-environment/tools/)
  to learn tips for production-quality deployments using each of those deployment
  methods. Different [Container Runtimes](/docs/setup/production-environment/container-runtimes/)
  are available to use with your deployments.
- *Manage certificates*: Secure communications between control plane services
  are implemented using certificates. Certificates are automatically generated
  during deployment or you can generate them using your own certificate authority.
  See [PKI certificates and requirements](/docs/setup/best-practices/certificates/) for details.
- *Configure load balancer for apiserver*: Configure a load balancer
  to distribute external API requests to the apiserver service instances running on different nodes. See 
  [Create an External Load Balancer](/docs/tasks/access-application-cluster/create-external-load-balancer/)
  for details.
- *Separate and backup etcd service*: The etcd services can either run on the
  same machines as other control plane services or run on separate machines, for
  extra security and availability. Because etcd stores cluster configuration data,
  backing up the etcd database should be done regularly to ensure that you can
  repair that database if needed.
  See the [etcd FAQ](https://etcd.io/docs/v3.5/faq/) for details on configuring and using etcd.
  See [Operating etcd clusters for Kubernetes](/docs/tasks/administer-cluster/configure-upgrade-etcd/)
  and [Set up a High Availability etcd cluster with kubeadm](/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/)
  for details.
- *Create multiple control plane systems*: For high availability, the
  control plane should not be limited to a single machine. If the control plane
  services are run by an init service (such as systemd), each service should run on at
  least three machines. However, running control plane services as pods in
  Kubernetes ensures that the replicated number of services that you request
  will always be available.
  The scheduler should be fault tolerant,
  but not highly available. Some deployment tools set up [Raft](https://raft.github.io/)
  consensus algorithm to do leader election of Kubernetes services. If the
  primary goes away, another service elects itself and take over. 
- *Span multiple zones*: If keeping your cluster available at all times is
  critical, consider creating a cluster that runs across multiple data centers,
  referred to as zones in cloud environments. Groups of zones are referred to as regions.
  By spreading a cluster across
  multiple zones in the same region, it can improve the chances that your
  cluster will continue to function even if one zone becomes unavailable.
  See [Running in multiple zones](/docs/setup/best-practices/multiple-zones/) for details.
- *Manage on-going features*: If you plan to keep your cluster over time,
  there are tasks you need to do to maintain its health and security. For example,
  if you installed with kubeadm, there are instructions to help you with
  [Certificate Management](/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)
  and [Upgrading kubeadm clusters](/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/).
  See [Administer a Cluster](/docs/tasks/administer-cluster/)
  for a longer list of Kubernetes administrative tasks.

To learn about available options when you run control plane services, see
[kube-apiserver](/docs/reference/command-line-tools-reference/kube-apiserver/),
[kube-controller-manager](/docs/reference/command-line-tools-reference/kube-controller-manager/),
and [kube-scheduler](/docs/reference/command-line-tools-reference/kube-scheduler/)
component pages. For highly available control plane examples, see
[Options for Highly Available topology](/docs/setup/production-environment/tools/kubeadm/ha-topology/),
[Creating Highly Available clusters with kubeadm](/docs/setup/production-environment/tools/kubeadm/high-availability/),
and [Operating etcd clusters for Kubernetes](/docs/tasks/administer-cluster/configure-upgrade-etcd/).
See [Backing up an etcd cluster](/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)
for information on making an etcd backup plan.

### Production worker nodes

Production-quality workloads need to be resilient and anything they rely
on needs to be resilient (such as CoreDNS). Whether you manage your own
control plane or have a cloud provider do it for you, you still need to
consider how you want to manage your worker nodes (also referred to
simply as *nodes*).  

- *Configure nodes*: Nodes can be physical or virtual machines. If you want to
  create and manage your own nodes, you can install a supported operating system,
  then add and run the appropriate
  [Node services](/docs/concepts/overview/components/#node-components). Consider:
  - The demands of your workloads when you set up nodes by having appropriate memory, CPU, and disk speed and storage capacity available.
  - Whether generic computer systems will do or you have workloads that need GPU processors, Windows nodes, or VM isolation.
- *Validate nodes*: See [Valid node setup](/docs/setup/best-practices/node-conformance/)
  for information on how to ensure that a node meets the requirements to join
  a Kubernetes cluster.
- *Add nodes to the cluster*: If you are managing your own cluster you can
  add nodes by setting up your own machines and either adding them manually or
  having them register themselves to the cluster’s apiserver. See the
  [Nodes](/docs/concepts/architecture/nodes/) section for information on how to set up Kubernetes to add nodes in these ways.
- *Scale nodes*: Have a plan for expanding the capacity your cluster will
  eventually need. See [Considerations for large clusters](/docs/setup/best-practices/cluster-large/)
  to help determine how many nodes you need, based on the number of pods and
  containers you need to run. If you are managing nodes yourself, this can mean
  purchasing and installing your own physical equipment.
- *Autoscale nodes*: Most cloud providers support
  [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler#readme)
  to replace unhealthy nodes or grow and shrink the number of nodes as demand requires. See the
  [Frequently Asked Questions](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md)
  for how the autoscaler works and
  [Deployment](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler#deployment)
  for how it is implemented by different cloud providers. For on-premises, there
  are some virtualization platforms that can be scripted to spin up new nodes
  based on demand.
- *Set up node health checks*: For important workloads, you want to make sure
  that the nodes and pods running on those nodes are healthy. Using the
  [Node Problem Detector](/docs/tasks/debug/debug-cluster/monitor-node-health/)
  daemon, you can ensure your nodes are healthy.

## Production user management

In production, you may be moving from a model where you or a small group of
people are accessing the cluster to where there may potentially be dozens or
hundreds of people. In a learning environment or platform prototype, you might have a single
administrative account for everything you do. In production, you will want
more accounts with different levels of access to different namespaces.

Taking on a production-quality cluster means deciding how you
want to selectively allow access by other users. In particular, you need to
select strategies for validating the identities of those who try to access your
cluster (authentication) and deciding if they have permissions to do what they
are asking (authorization):

- *Authentication*: The apiserver can authenticate users using client
  certificates, bearer tokens, an authenticating proxy, or HTTP basic auth.
  You can choose which authentication methods you want to use.
  Using plugins, the apiserver can leverage your organization’s existing
  authentication methods, such as LDAP or Kerberos. See
  [Authentication](/docs/reference/access-authn-authz/authentication/)
  for a description of these different methods of authenticating Kubernetes users.
- *Authorization*: When you set out to authorize your regular users, you will probably choose
  between RBAC and ABAC authorization. See [Authorization Overview](/docs/reference/access-authn-authz/authorization/)
  to review different modes for authorizing user accounts (as well as service account access to
  your cluster):
  - *Role-based access control* ([RBAC](/docs/reference/access-authn-authz/rbac/)): Lets you
    assign access to your cluster by allowing specific sets of permissions to authenticated users.
    Permissions can be assigned for a specific namespace (Role) or across the entire cluster
    (ClusterRole). Then using RoleBindings and ClusterRoleBindings, those permissions can be attached
    to particular users.
  - *Attribute-based access control* ([ABAC](/docs/reference/access-authn-authz/abac/)): Lets you
    create policies based on resource attributes in the cluster and will allow or deny access
    based on those attributes. Each line of a policy file identifies versioning properties (apiVersion
    and kind) and a map of spec properties to match the subject (user or group), resource property,
    non-resource property (/version or /apis), and readonly. See
    [Examples](/docs/reference/access-authn-authz/abac/#examples) for details.

As someone setting up authentication and authorization on your production Kubernetes cluster, here are some things to consider:

- *Set the authorization mode*: When the Kubernetes API server
  ([kube-apiserver](/docs/reference/command-line-tools-reference/kube-apiserver/))
  starts, the supported authentication modes must be set using the *--authorization-mode*
  flag. For example, that flag in the *kube-adminserver.yaml* file (in */etc/kubernetes/manifests*)
  could be set to Node,RBAC. This would allow Node and RBAC authorization for authenticated requests.
- *Create user certificates and role bindings (RBAC)*: If you are using RBAC
  authorization, users can create a CertificateSigningRequest (CSR) that can be
  signed by the cluster CA. Then you can bind Roles and ClusterRoles to each user.
  See [Certificate Signing Requests](/docs/reference/access-authn-authz/certificate-signing-requests/)
  for details.
- *Create policies that combine attributes (ABAC)*: If you are using ABAC
  authorization, you can assign combinations of attributes to form policies to
  authorize selected users or groups to access particular resources (such as a
  pod), namespace, or apiGroup. For more information, see
  [Examples](/docs/reference/access-authn-authz/abac/#examples).
- *Consider Admission Controllers*: Additional forms of authorization for
  requests that can come in through the API server include
  [Webhook Token Authentication](/docs/reference/access-authn-authz/authentication/#webhook-token-authentication).
  Webhooks and other special authorization types need to be enabled by adding
  [Admission Controllers](/docs/reference/access-authn-authz/admission-controllers/)
  to the API server.

## Set limits on workload resources

Demands from production workloads can cause pressure both inside and outside
of the Kubernetes control plane. Consider these items when setting up for the
needs of your cluster's workloads:

- *Set namespace limits*: Set per-namespace quotas on things like memory and CPU. See
  [Manage Memory, CPU, and API Resources](/docs/tasks/administer-cluster/manage-resources/)
  for details. You can also set
  [Hierarchical Namespaces](/blog/2020/08/14/introducing-hierarchical-namespaces/)
  for inheriting limits.
- *Prepare for DNS demand*: If you expect workloads to massively scale up,
  your DNS service must be ready to scale up as well. See
  [Autoscale the DNS service in a Cluster](/docs/tasks/administer-cluster/dns-horizontal-autoscaling/).
- *Create additional service accounts*: User accounts determine what users can
  do on a cluster, while a service account defines pod access within a particular
  namespace. By default, a pod takes on the default service account from its namespace.
  See [Managing Service Accounts](/docs/reference/access-authn-authz/service-accounts-admin/)
  for information on creating a new service account. For example, you might want to:
  - Add secrets that a pod could use to pull images from a particular container registry. See
    [Configure Service Accounts for Pods](/docs/tasks/configure-pod-container/configure-service-account/)
    for an example.
  - Assign RBAC permissions to a service account. See
    [ServiceAccount permissions](/docs/reference/access-authn-authz/rbac/#service-account-permissions)
    for details.

## {{% heading "whatsnext" %}}

- Decide if you want to build your own production Kubernetes or obtain one from
  available [Turnkey Cloud Solutions](/docs/setup/production-environment/turnkey-solutions/)
  or [Kubernetes Partners](/partners/).
- If you choose to build your own cluster, plan how you want to
  handle [certificates](/docs/setup/best-practices/certificates/)
  and set up high availability for features such as
  [etcd](/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/)
  and the
  [API server](/docs/setup/production-environment/tools/kubeadm/ha-topology/).
- Choose from [kubeadm](/docs/setup/production-environment/tools/kubeadm/),
  [kops](/docs/setup/production-environment/tools/kops/) or
  [Kubespray](/docs/setup/production-environment/tools/kubespray/)
  deployment methods.
- Configure user management by determining your
  [Authentication](/docs/reference/access-authn-authz/authentication/) and
  [Authorization](/docs/reference/access-authn-authz/authorization/) methods.
- Prepare for application workloads by setting up
  [resource limits](/docs/tasks/administer-cluster/manage-resources/),
  [DNS autoscaling](/docs/tasks/administer-cluster/dns-horizontal-autoscaling/)
  and [service accounts](/docs/reference/access-authn-authz/service-accounts-admin/).

