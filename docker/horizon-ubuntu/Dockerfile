FROM ubuntu
MAINTAINER Ivan Mikushin (https://github.com/imikushin)

RUN apt-get install -y openstack-dashboard apache2 libapache2-mod-wsgi python-memcache && \
    apt-get remove -y --auto-remove openstack-dashboard-ubuntu-theme

RUN sed -i.bak 's/django.core.cache.backends.memcached.MemcachedCache/django.core.cache.backends.locmem.LocMemCache/' \
    /etc/openstack-dashboard/local_settings.py

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
