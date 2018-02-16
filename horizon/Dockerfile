FROM ubuntu:16.04

ENV MON_KEYSTONE_URL=127.0.0.1:5000 \
 GRAFANA_URL=127.0.0.1:3000

RUN apt-get -y update && apt-get -y upgrade
RUN apt-get -y install \
        git \
	python-pip \
	apache2 \
	libapache2-mod-wsgi \
	&& apt-get clean
RUN pip install shinto-cli

COPY start.sh local_settings.py.j2 monitoring-local_settings.py.j2 /

RUN git clone --depth=1 https://git.openstack.org/openstack/monasca-ui -b stable/ocata
RUN pip install -c http://git.openstack.org/cgit/openstack/requirements/plain/upper-constraints.txt?h=stable/ocata monasca-ui/.

RUN git clone https://git.openstack.org/openstack/horizon -b 10.0.5 --depth 1
WORKDIR horizon
RUN pip install -c http://git.openstack.org/cgit/openstack/requirements/plain/upper-constraints.txt?h=stable/newton .


RUN ln -sf /monasca-ui/monitoring/enabled/_50_admin_add_monitoring_panel.py \
/horizon/openstack_dashboard/enabled/_50_admin_add_monitoring_panel.py && \
ln -sf /monasca-ui/monitoring/conf/monitoring_policy.json \
/horizon/openstack_dashboard/conf/monitoring_policy.json && \
ln -sfF /monasca-ui/monitoring /horizon/monitoring

RUN ./manage.py collectstatic --noinput && ./manage.py make_web_conf --wsgi && \
 ./manage.py make_web_conf --apache > /etc/apache2/sites-available/horizon.conf

RUN chown -R www-data:www-data  /horizon/openstack_dashboard/local/.secret_key_store /horizon/static/dashboard

RUN a2ensite horizon && a2dissite 000-default

EXPOSE 80 
CMD /start.sh

healthcheck --interval=10s --timeout=30s --start-period=30s \
  CMD curl -f http://localhost/  || exit 1;
