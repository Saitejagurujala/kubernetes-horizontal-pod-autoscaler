Run and expose php-apache server
To demonstrate a HorizontalPodAutoscaler, you will first make a custom container image that uses the php-apache image from Docker Hub as its starting point. The Dockerfile is ready-made for you, and has the following content:

FROM php:5-apache
COPY index.php /var/www/html/index.php
RUN chmod a+rx index.php
This code defines a simple index.php page that performs some CPU intensive computations, in order to simulate load in your cluster.

<?php
  $x = 0.0001;
  for ($i = 0; $i <= 1000000; $i++) {
    $x += sqrt($x);
  }
  echo "OK!";
?>


Now run in another terminal:

# type Ctrl+C to end the watch when you're ready

kubectl get hpa php-apache --watch

----------Load Generator --------------------

Increase the load
Next, see how the autoscaler reacts to increased load. To do this, you'll start a different Pod to act as a client. The container within the client Pod runs in an infinite loop, sending queries to the php-apache service.


# Run this in a separate terminal
# so that the load generation continues and you can carry on with the rest of the steps


kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"