FROM sameersbn/squid
RUN apt update && DEBIAN_FRONTEND=noninteractive apt install krb5-user msktutil -y
ADD krb5.conf /etc/
ADD HTTP.keytab /etc/squid
ADD squid.conf /etc/squid
ENV KRB5_KTNAME=/etc/squid/HTTP.keytab
RUN chgrp proxy /etc/squid/HTTP.keytab && chmod g+r /etc/squid/HTTP.keytab