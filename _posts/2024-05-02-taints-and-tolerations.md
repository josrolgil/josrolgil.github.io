---
layout: post
title: Kubernetes taints and tolerations
excerpt_separator: <!--preview-->
---
The objective of this article is to summarize one mechanism available in Kubernetes to manage pods scheduling, called taints and tolerations.
<!--preview-->

Taints are applied to nodes. Basically, once you taint the node, by default it applies the effect to all pods, unless the pod tolerates the taint.
A taint consists of a key, a value and an effect. An example is: key1=value1:NoSchedule. This can be executed with kubectl command.

*kubectl taint nodes node1 key1=value1:NoSchedule*

What are the available effects? There you have:
- NoExecute: applies to already running pods, which will be evicted unless the pods tolerates it.
- NoSchedule: applies to new pods, which will not be schedule on the node unless tolerate the taint.
- PreferNoSchedule: same as NoSchedule, but is not guaranteed that the pod will not be scheduled. I understand that there could be conflicts with other configurations and in some cases this taint does not apply.

![_config.yml]({{ site.baseurl }}/images/2024/tolerations/tolerations.PNG){: loading="lazy"}

As was already commented, in order to ignore the effect, a pod can define tolerations in is definition. There are two forms of define a toleration, which I define as narrow or wide.
Regarding narrow, a pod will indicate specifically the key, value and effect that it tolerates. For that, the operator "Equal" is used. For example:

```yaml
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
```
Regarding wide, the toleration only specify the key, and matches all values. For example:
```yaml
tolerations:
- key: "key1"
  operator: "Exists"
  effect: "NoSchedule"
```
Please note that the effect has to match also when evaluating taints and tolerations.

We can say that this mechanism is an "allowlist" mechanism where, by default, all pods will be evicted or no scheduled unless it is specify the contrary on some pods with tolerations.

The official documentation is [here](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/).
