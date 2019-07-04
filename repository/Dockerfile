FROM ubuntu:18.04

# MAINTAINER Yinlin Chen "ylchen@vt.edu"
MAINTAINER Dan Kerchner "kerchner@gwu.edu"

ARG FedoraConfig=
ARG ModeshapeConfig=file-simple

# Install essential packages
RUN apt-get update && apt-get install -y \
        lsof \
        unzip \
	build-essential \
	curl \
	maven \
	openssh-server \
	software-properties-common \
	vim \
	wget \
	htop tree zsh fish \
        openjdk-8*


# Install Java 8
# Define commonly used JAVA_HOME variable
ENV JAVA_HOME /usr/lib/jvm/java-8-oracle

# Install Tomcat 7.
ENV CATALINA_HOME /usr/local/tomcat7
ENV PATH $CATALINA_HOME/bin:$PATH
ENV TOMCAT_MAJOR 7
ENV TOMCAT_VERSION 7.0.94
ENV TOMCAT_TGZ_URL http://archive.apache.org/dist/tomcat/tomcat-$TOMCAT_MAJOR/v$TOMCAT_VERSION/bin/apache-tomcat-$TOMCAT_VERSION.tar.gz

RUN mkdir -p "$CATALINA_HOME" \
	&& cd $CATALINA_HOME \
	&& set -x \
	&& curl -fSL "$TOMCAT_TGZ_URL" -o tomcat.tar.gz \
	&& tar -xvf tomcat.tar.gz --strip-components=1 \
	&& rm bin/*.bat \
	&& rm tomcat.tar.gz* \
	&& useradd -ms /bin/bash tomcat7 \
	&& sed -i '$i<role rolename="fedoraUser"/>$i<role rolename="fedoraAdmin"/>$i<role rolename="manager-gui"/>$i<user username="testuser" password="password1" roles="fedoraUser"/>$i<user username="adminuser" password="password2" roles="fedoraUser"/>$i<user username="fedoraAdmin" password="secret3" roles="fedoraAdmin"/>$i<user username="fedora4" password="fedora4" roles="manager-gui"/>' /usr/local/tomcat7/conf/tomcat-users.xml

RUN echo 'JAVA_OPTS="$JAVA_OPTS -Dfcrepo.modeshape.configuration=classpath:/config/'$ModeshapeConfig'/repository.json -Dfcrepo.home=/mnt/ingest -Dfcrepo.audit.container=/audit"' > $CATALINA_HOME/bin/setenv.sh \
	&& chmod +x $CATALINA_HOME/bin/setenv.sh


# Make the ingest directory
RUN mkdir /mnt/ingest \
	&& chown -R tomcat7:tomcat7 /mnt/ingest

VOLUME /mnt/ingest


# Install Fedora4
ENV FEDORA_VERSION 4.7.5
ENV FEDORA_TAG 4.7.5

RUN mkdir -p /var/lib/tomcat7/fcrepo4-data \
	&& chown tomcat7:tomcat7 /var/lib/tomcat7/fcrepo4-data \
	&& chmod g-w /var/lib/tomcat7/fcrepo4-data \
	&& cd /tmp \
	&& curl -fSL https://github.com/fcrepo4-exts/fcrepo-webapp-plus/releases/download/fcrepo-webapp-plus-$FEDORA_TAG/fcrepo-webapp-plus-$FedoraConfig$FEDORA_VERSION.war -o fcrepo.war \
	&& cp fcrepo.war /usr/local/tomcat7/webapps/fcrepo.war \
	&& chown tomcat7:tomcat7 /usr/local/tomcat7/webapps/fcrepo.war 


# Install Solr
ENV SOLR_VERSION 6.4.1
ENV SOLR_HOME /usr/local/tomcat7/solr

RUN cd /tmp \
	&& curl -fSL http://repo1.maven.org/maven2/commons-logging/commons-logging/1.1.2/commons-logging-1.1.2.jar -o commons-logging-1.1.2.jar 

RUN mkdir -p /opt/install \
        && cd /opt/install \
	&& curl -fSL http://archive.apache.org/dist/lucene/solr/$SOLR_VERSION/solr-$SOLR_VERSION.tgz -o solr-$SOLR_VERSION.tgz \
	&& mkdir -p "$SOLR_HOME" \
	&& tar -xzf solr-"$SOLR_VERSION".tgz solr-"$SOLR_VERSION"/bin/install_solr_service.sh --strip-components=2 \
        && ./install_solr_service.sh solr-"$SOLR_VERSION".tgz \
        && service solr start \
        && su - solr -c "/opt/solr/bin/solr create -c scholarspace -n /opt/install/solr/config" \
        && mv /var/solr/data/scholarspace/conf/managed-schema /var/solr/data/scholarspace/conf/managed-schema.bak 

RUN cd /tmp \
        && curl -fSL https://github.com/samvera/hyrax/archive/v2.0.3.zip -o hyrax.zip \
        && unzip hyrax.zip \
        && cp -r /tmp/hyrax-2.0.3/solr/config/* /var/solr/data/scholarspace/conf/ \
        && service solr restart


CMD tail -f /dev/null
