---
layout: post
title:  "Kubernetes container probes"
date:   2022-08-19 19:19:05 +0530
categories: jekyll update
---
The container probes are among the basic best practices advised for k8s
cluster before going to production. It helps for deploying self probing and resilient services(restart) on Kubernetes. 							
            
The differnt categories under which we can put k8s container probes can
be broadly as follows:

- `readiness-probe`
- `liveness-probe`

1. Readiness Probe
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

For further reads on different types of checks configurable and a deep dive please visit the [kubernets official documentation pages](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/).







Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/ 
