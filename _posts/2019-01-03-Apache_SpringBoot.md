---
layout: post
title: Apache-SpringBoot 연동
date: 2019-01-03 15:10
categories: Etc
---

# Apache - SpringBoot(내장톰캣) 연동

### 개요

`사내정보 시스템 SSO`연동을 위해 Apache사용이 불가피함


### Part1. mod_jk를 이용한 tomcat 연동(Apache설정)

사내에서 발급받은 서버에는 기본적으로 Apahce2.2가 설치되어 있어 Apahce설치과정은 생략한다.

mod_jk를 설치과정 및 Apache설정은 다음과 같다.

---

1) **tomcat-connectors설치**

```bash
cd /usr/local/src
wget http://www.apache.org/dist/tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.44-src.tar.gz
tar -xzf tomcat-connectors-1.2.44-src.tar.gz
```

2) **native 디렉토리로 이동**

```bash
cd tomcat-connectors-1.2.44-src/native
```

3) **컴파일**

```bash
./configure --with-apxs=/usr/sbin/apxs
make
make install
```

apxs path를 입력해줘야하는데, defulat는 `/usr/sbin/apxs`이다. 만약 이 경로에 없다면 찾아서 알맞게 입력하자.

만약 없는 경우

```bash
sudo yum install httpd-devel
```

4) **httpd.conf수정**

```bash
cd /etc/httpd/conf
sudo vi httpd.conf
```

(httpd.conf) 맨아래추가

```bash
LoadModule jk_module modules/mod_jk.so
include conf/mod_jk.conf
include conf/http_vhost.conf
```

5) **mod_jk.conf 추가**

```bash
vi mod_jk.conf
```
(mod_jk.conf)
```bash
<IfModule mod_jk.c>
 JkWorkersFile conf/workers.properties
 JkLogFile     logs/mod_jk.log
 JkLogLevel    info
</IfModule>
```

6) **workers.properties 추가**

```bash
vi worker.properties
```

(workers.properties)

```bash
worker.list=worker1,worker2
 
worker.worker1.port=18009
worker.worker1.host=127.0.0.1
worker.worker1.type=ajp13
worker.worker1.lbfactor=1

worker.worker2.port=28009
worker.worker2.host=127.0.0.1
worker.worker2.type=ajp13
worker.worker2.lbfactor=1
```

work가 여러개이면 woker.list에 콤마(,)구분자로 추가

7) **http_vhost.conf** 추가

```bash
vi http_vhost.conf
```

(http_vhost.conf)

```bash
<VirtualHost *:80>
 ServerName  customtargeting.nhnent.com
 JkMount /* worker1
</VirtualHost>

<VirtualHost *:80>
 ServerName  addinfra-site.nhnent.com
 JkMount /* worker2
 </VirtualHost>
```

서비스를 추가할려면 `workers.properties`와 `http_vhost.conf`에 값을 추가하고 apache를 재시작해주면 된다.


### Part2. mod_jk를 이용한 tomcat 연동(Spring-Boot설정)

SpringBoot 1.x와 SpringBoot 2.x의 AJP포트 설정 법에 차이가 있다.
그 이유는 SpringBoot1.x에서는 내장톰캣을 기본으로 사용하고 있었지만 SpringBoot2.x에서는 내장톰캣대신에 netty를 사용하기 때문이다.

1)  **application.properties 값추가**

```
tomcat.ajp.protocol=AJP/1.3
tomcat.ajp.port=18009
tomcat.ajp.enabled=true
```

2-1) **SpringBoot1.x ContainerConfig 클래스 추가**

```java
import org.apache.catalina.connector.Connector;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.embedded.EmbeddedServletContainerCustomizer;
import org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ContainerConfig {
	@Value("${tomcat.ajp.protocol}")
	String ajpProtocol;

	@Value("${tomcat.ajp.port}")
	int ajpPort;

	@Value("${tomcat.ajp.enabled}")
	boolean tomcatAjpEnabled;

	@Bean
	public EmbeddedServletContainerCustomizer containerCustomizer() {
		return container -> {
			TomcatEmbeddedServletContainerFactory tomcat = (TomcatEmbeddedServletContainerFactory) container;
			if (tomcatAjpEnabled) {
				Connector ajpConnector = new Connector(ajpProtocol);
				ajpConnector.setPort(ajpPort);
				ajpConnector.setSecure(false);
				ajpConnector.setAllowTrace(false);
				ajpConnector.setScheme("http");
				tomcat.addAdditionalTomcatConnectors(true);
			}
		};
	}
}
```

2-2) **SpringBoot2.x ContainerConfig 클래스 추가**

```java
import org.apache.catalina.connector.Connector;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.boot.web.servlet.server.ServletWebServerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ContainerConfig {
	@Value("${tomcat.ajp.protocol}")
	String ajpProtocol;

	@Value("${tomcat.ajp.port}")
	int ajpPort;

	@Bean
	public ServletWebServerFactory servletContainer() {
		TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
		tomcat.addAdditionalTomcatConnectors(createAjpConnector());
		return tomcat;
	}

	private Connector createAjpConnector() {
		Connector ajpConnector = new Connector(ajpProtocol);
		ajpConnector.setPort(ajpPort);
		ajpConnector.setSecure(false);
		ajpConnector.setAllowTrace(false);
		ajpConnector.setScheme("http");
		return ajpConnector;
	}
}
```

3) 부팅 로그 확인

```bash
10:50:32.520 [main] [INFO ] o.a.coyote.http11.Http11NioProtocol - Initializing ProtocolHandler ["http-nio-8080"]
10:50:32.538 [main] [INFO ] o.a.coyote.http11.Http11NioProtocol - Starting ProtocolHandler ["http-nio-8080"]
10:50:32.544 [main] [INFO ] o.a.tomcat.util.net.NioSelectorPool - Using a shared selector for servlet write/read
10:50:32.560 [main] [INFO ] o.apache.coyote.ajp.AjpNioProtocol - Initializing ProtocolHandler ["ajp-nio-18009"]
10:50:32.563 [main] [INFO ] o.a.tomcat.util.net.NioSelectorPool - Using a shared selector for servlet write/read
10:50:32.563 [main] [INFO ] o.apache.coyote.ajp.AjpNioProtocol - Starting ProtocolHandler ["ajp-nio-18009"]
10:50:32.576 [main] [INFO ] o.s.b.c.e.t.TomcatEmbeddedServletContainer - Tomcat started on port(s): 8080 (http) 18009 (http)
```

ajp포트가 올라왔는지 확인한다.
