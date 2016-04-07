# Use phusion/baseimage as base image. To make your builds
# reproducible, make sure you lock down to a specific version, not
# to `latest`! See
# https://github.com/phusion/baseimage-docker/blob/master/Changelog.md
# for a list of version numbers.
FROM phusion/baseimage:0.9.18

# Environment Variables
ENV DNSDOMAIN '.mydomain.local' # Make sure to add a leading dot ex: .home.local
ENV SNMPCOMMUNITY 'public'
ENV DEBIAN_FRONTEND=noninteractive

# Use baseimage-docker's init system.
CMD ["/sbin/my_init"]
RUN apt-get update
RUN apt-get install -y wget apache2 libsnmp-perl libapache2-mod-perl2 libnet-snmp-perl libsnmp-info-perl
WORKDIR /opt
RUN wget https://sourceforge.net/projects/switchmap/files/latest/download?source=files -O switchmap.tar.gz
RUN mkdir switchmap
RUN tar zxvf switchmap.tar.gz -C switchmap --strip-components 1
RUN mkdir -p /opt/switchmap/web
RUN ln -s /opt/switchmap/web /var/www/html/switchmap
WORKDIR /opt/switchmap
RUN cp ThisSite.pm ThisSite.pm.orig
# Fix Configuration Files
RUN sed -i "s/^\$Community \= 'public';$/\$Community \= '${SNMPCOMMUNITY}';/m" ThisSite.pm
RUN sed -i "s/^\$UseSysNames \= 0;$/\$UseSysNames \= 1;/m" ThisSite.pm
RUN sed -i "s/^\$ShowCdpName \= 0;$/\$ShowCdpName \= 1;/m" ThisSite.pm
RUN sed -i "s/^\$DnsDomain \= '\.your\.domain';$/\$DnsDomain \= '${DNSDOMAIN}';/m" ThisSite.pm
RUN sed -i "s/^\$DestinationDirectory \= '';$/\$DestinationDirectory \= '\/opt\/switchmap\/web';/m" ThisSite.pm
RUN sed -i "s/^\$DestinationDirectoryRoot \= '';$/\$DestinationDirectoryRoot \= '\/switchmap';/m" ThisSite.pm
RUN sed -i "s/^\$StateFileDirectory \= '\/var\/local\/switchmap';$/\$StateFileDirectory \= '\/var\/run\/switchmap';/m" ThisSite.pm
RUN sed -i "s/^@routers/#@routers/m" ThisSite.pm
RUN sed -i "s/push @routers/#push @routers/m" ThisSite.pm
RUN sed -i "s/^@LocalSwitches/#@LocalSwitches/m" ThisSite.pm
RUN sed -i "s/^push @LocalSwitches/#push @LocalSwitches/m" ThisSite.pm
# Fix HTML Files
RUN sed -i "s/\/nets\/internal\/portlists\//\/opt\/switchmap\/web\//m" SearchPortlists.html
RUN sed -i "s/\/nets\/cgi\/internal\//\/cgi-bin\//m" SearchPortlists.html
RUN a2enmod ssl
RUN sed -i "s/#Include conf-available\/serve-cgi-bin\.conf/Include conf-available\/serve-cgi-bin\.conf/m" /etc/apache2/sites-available/default-ssl.conf
RUN a2enmod cgi
RUN ln -s /opt/switchmap/FindOffice.pl /usr/lib/cgi-bin/
RUN ln -s /opt/switchmap/SearchPortLists.html /opt/switchmap/web/
RUN sed -i "s/\/usr\/web\/nets\/internal\/portlists/\/opt\/switchmap/m" FindOffice.pl
RUN sed -i "s/+SymLinksIfOwnerMatch/+FollowSymLinks/m" /etc/apache2/conf-available/serve-cgi-bin.conf
RUN sed -i "s/\/usr\/lib\/cgi-bin/\/opt\/switchmap/m" /etc/apache2/conf-available/serve-cgi-bin.conf
RUN sed -i "s/\/usr\/lib\/cgi-bin/\/opt\/switchmap/m" /etc/apache2/sites-available/default-ssl.conf
# Make Configs Directory
RUN mkdir /configs
RUN ln -s /opt/switchmap/ThisSite.pm /configs/
VOLUME ["/configs"]
# Enable Sites
RUN find /etc/apache2/sites-enabled/ -type l -exec rm -f "{}" \;
RUN a2ensite default-ssl
#RUN chown www-data /opt/switchmap/FindOffice.pl
RUN echo '30 * * * * root perl /opt/switchmap/ScanSwitch.pl\n50 * * * * root perl /opt/switchmap/GetArp.pl\n05 14 * * * root perl /opt/switchmap/SwitchMap.pl\n' >> /etc/crontab
RUN mkdir -p /etc/service/apache2
RUN echo '#!/bin/sh\nexec apache2ctl -D FOREGROUND -k start' > /etc/service/apache2/run
RUN chmod 755 /etc/service/apache2/run
# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
