---
layout: post
title:  "Kubernetes container probes"
date:   2022-05-06 11:20:05 +0530
categories: jekyll update
---
The container probes are among the basic best practices advised for k8s
cluster before going to production. It helps for deploying self probing and resilient services(restart) on Kubernetes. 							
            
The different categories under which we can put k8s container probes can
be broadly as follows:

- `readiness-probe`
- `liveness-probe`

## 1. Readiness Probe
The readiness probe determines when a container can receive
                  traffic.

The kubelet executes the checks and decides if the app is ready to receive traffic or not.
Hence the name *readiness*.

>**Note:** Containers are always ready which is not always desirable.

There's no default value for readiness and
                  liveness.

If you don't set the readiness probe, the kubelet assumes that
                  the app is ready to receive traffic as soon as the container
                  starts.

If the container takes 2 minutes to start, all the requests to
                  it will fail for those 2 minutes.

Lets have a quick look at how we applied our config


{% highlight yaml %}
readinessProbe:
  failureThreshold: 3
  initialDelaySeconds: 5
  periodSeconds: 5
  successThreshold: 1
  tcpSocket:
    port: 80
  timeoutSeconds: 1
{% endhighlight %}

Generally we can translate the yaml configuration as the `readiness probe` waits for 5 seconds on container initializaton and then onwards the probes checks the health of *tcp/port 80* of the container on an interval of 5 sec. 

>**Note:** Readiness probes runs on the container during its whole lifecycle.

If the probe sends `OK` response then it is considered healthy. If it ends up in 3 consecutive failure, the container will be considered unhealthy and all traffic to the container will be stopped.

## 2.The liveness probes

Containers crash when there's a fatal error

Lets look at a typical liveness probe config yaml:

```yaml
livenessProbe:
  failureThreshold: 3
  initialDelaySeconds: 15
  periodSeconds: 10
  successThreshold: 1
  tcpSocket:
    port: 80
  timeoutSeconds: 1
```

If the application reaches an unrecoverable error, [you should let it crash](https://blog.colinbreck.com/kubernetes-liveness-and-readiness-probes-revisited-how-to-avoid-shooting-yourself-in-the-other-foot/#letitcrash)

Examples of such unrecoverable errors are:
- an uncaught exception
- a typo in the code (for dynamic languages)
- unable to load a header or dependency

Please note that you should not signal a failing Liveness
                  probe.

Instead, you should immediately exit the process and let the
                  kubelet restart the container.

The liveness probe determines when a container should be
                  restarted.

The kubelet executes the check and decides if the container
                  should be restarted.


For further reads on different types of checks configurable and a deep dive please visit the [kubernets official documentation pages](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/).

> ### Resources:
- The official Kubernetes documentation offers some practical
                    advice on how to [configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [Liveness probes are dangerous](https://srcco.de/posts/kubernetes-liveness-probes-are-dangerous.html) has some information on how to set (or not) dependencies in your readiness probes.





Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/ 
