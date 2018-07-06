---
layout: post
title: "Jms-config с ssl"
permalink: jms-config-with-ssl
tags: ibm-mq ssl windows java jms spring
---
Для работы с SSL в java-приложении необходимо настроить соответствующим образом фабрику соединений.
- Создадим утилитный класс, в котором настроим **SSLContext**, для которого необходимо передать **пути к файлам и пароли к keyStore и trustStore.**

		import javax.net.ssl.KeyManagerFactory;  
		import javax.net.ssl.SSLContext;  
		import javax.net.ssl.TrustManagerFactory;  
		import java.io.FileInputStream; 
		import java.io.IOException;  
		import java.security.*;  

		public class KeyUtils {  
		  
			public static KeyStore createKeystoreFromJKS(String jksPath, String ksPassword) {  
				try {  
					KeyStore keyStore = KeyStore.getInstance("JKS");  
					keyStore.load(new FileInputStream(jksPath), ksPassword.toCharArray());  
					return keyStore;  
				} catch (GeneralSecurityException | IOException e) {  
					throw new RuntimeException(e);  
				}  
			}  
			  
			public static SSLContext createSSLContext(String keyStore, String keyStorePassword, String trustStore, String trustStorePassword) {  
				KeyStore ks = createKeystoreFromJKS(keyStore, keyStorePassword);  
				KeyStore ts = createKeystoreFromJKS(trustStore, trustStorePassword);  
				  
				try {  
					KeyManagerFactory kmf = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());  
					kmf.init(ks, keyStorePassword.toCharArray());  
					  
					TrustManagerFactory tmf = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());  
					tmf.init(ts);  
					  
					SSLContext sslContext = SSLContext.getInstance("SSLv3");  
					sslContext.init(kmf.getKeyManagers(), tmf.getTrustManagers(), null);  
					return sslContext;  
				} catch (Exception e) {  
					throw new RuntimeException(e);  
				}  
			}  
		}

-   Пример класса конфигурации для JMS-контекста, использующего SSL

	    import com.ibm.mq.jms.MQQueueConnectionFactory;  
	    import com.ibm.msg.client.wmq.WMQConstants;  
	    import org.springframework.beans.factory.annotation.Value;  
	    import org.springframework.context.annotation.Bean;  
	    import org.springframework.context.annotation.Configuration; 
	    import org.springframework.context.annotation.Primary;
	    import org.springframework.jms.connection.CachingConnectionFactory;
	    import org.springframework.jms.core.JmsTemplate;  
		import javax.jms.JMSException;  
		import javax.jms.QueueConnectionFactory;  
		import javax.jms.Session;  
		import javax.net.ssl.SSLContext;
		import java.security.Security; 
        
        @Configuration
        public class JmsContext { 
			@Value("${servers.mq.host}")  
			private String host;  
			@Value("${servers.mq.port}")  
			private Integer port;  
			@Value("${servers.mq.queue-manager}")  
			private String queueManager;  
			@Value("${servers.mq.channel}")  
			private String channel;  
			@Value("${servers.mq.timeout}")  
			private long timeout;  
			@Value("${servers.mq.sessionCacheSize}")  
			private int sessionCacheSize;  
			@Value("${spring.application.name}")  
			private String appName;  
			@Value("${servers.mq.CCSID}")  
			private int ccsid;  
			@Value("${servers.mq.trustStore}")  
			private String trustStore;  
			@Value("${servers.mq.trustStorePassword}")  
			private String trustStorePassword;  
			@Value("${servers.mq.keyStore}")  
			private String keyStore;  
			@Value("${servers.mq.keyStorePassword}")  
			private String keyStorePassword;  
			@Value("${servers.mq.peerName}")  
			private String peerName;  
			@Value("${servers.mq.cipherSuite}")  
			private String cipherSuite;    
  
			@Bean  
			public QueueConnectionFactory queueConnectionFactory() throws JMSException {  
				Security.setProperty("jdk.tls.disabledAlgorithms", "");  
				SSLContext sslContext = KeyUtils.createSSLContext(keyStore, keyStorePassword,  trustStore,  trustStorePassword);  
				MQQueueConnectionFactory mqQueueConnectionFactory = new MQQueueConnectionFactory();
				mqQueueConnectionFactory.setHostName(host);
				mqQueueConnectionFactory.setQueueManager(queueManager);
				mqQueueConnectionFactory.setPort(port);
				mqQueueConnectionFactory.setChannel(channel);
				mqQueueConnectionFactory.setAppName(appName);
				mqQueueConnectionFactory.setTransportType(WMQConstants.WMQ_CM_CLIENT);
				mqQueueConnectionFactory.setCCSID(ccsid); 
				mqQueueConnectionFactory.setSSLCipherSuite(cipherSuite);
				mqQueueConnectionFactory.setSSLSocketFactory(sslContext.getSocketFactory());
				mqQueueConnectionFactory.setSSLPeerName(peerName);
				mqQueueConnectionFactory.setSSLFipsRequired(false);
				return mqQueueConnectionFactory;
		    }
		     
			@Bean  
			@Primary  
			public CachingConnectionFactory queueCachingConnectionFactory() throws JMSException {  
				CachingConnectionFactory cachingConnectionFactory = new CachingConnectionFactory();
				cachingConnectionFactory.setTargetConnectionFactory(queueConnectionFactory());
				cachingConnectionFactory.setSessionCacheSize(sessionCacheSize);
				cachingConnectionFactory.setReconnectOnException(true);  
			    return cachingConnectionFactory;  
			}  
			  
			@Bean  
			@Primary  
			public JmsTemplate queueTemplate(CachingConnectionFactory queueCachingConnectionFactory) {  
				JmsTemplate jmsTemplate = new JmsTemplate(queueCachingConnectionFactory);  
				jmsTemplate.setReceiveTimeout(timeout);  
				jmsTemplate.setSessionTransacted(true);  				jmsTemplate.setSessionAcknowledgeMode(Session.SESSION_TRANSACTED);  
				return jmsTemplate;  
			}  
		  
		}  
  
-   Пример для указания настроек JMS.

		servers:  
		mq:  
		host: 192.168.0.101  
		port: 1414  
		queue-manager: OUT.QM  
		channel: OUT.SVR.CONN  
		queue: OUT_XML  
		timeout: 2000  
		sessionCacheSize: 10  
		CCSID: 1208  
		trustStore: D:\192_168_0_101\trustStore.jks  
		trustStorePassword: 123456  
		keyStore: D:\192_168_0_101\keyStore.jks  
		keyStorePassword: 123456  
		peerName: CN=home.ofedorova.ru, OU=00CA, O=Home organization, C=RU  
		cipherSuite: SSL_RSA_WITH_RC4_128_MD5
