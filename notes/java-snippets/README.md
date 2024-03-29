# Java : fragments de code
## Autour des certificats
- Méthodes permettant d'extraire les certificats d'un _keystore_ et d'en créer un jouveau (fortement inspiré de [&#x21aa; wsargent/Directions for creating PEM files](https://gist.github.com/wsargent/26c0d478d0901d4b0a11eeb10dcbe0d1))
```java
private static final String SYSTEM_PROPERTY_SSL_TRUSTSTORE = "javax.net.ssl.trustStore";
private static final String SYSTEM_PROPERTY_SSL_TRUSTSTORE_PWD = "javax.net.ssl.trustStorePassword";
private static final String SYSTEM_PROPERTY_SSL_TRUSTSTORE_TYPE = "javax.net.ssl.trustStoreType";
private static final String SYSTEM_PROPERTY_SSL_KEYSTORE = "javax.net.ssl.keyStore";
private static final String SYSTEM_PROPERTY_SSL_KEYSTORE_PWD = "javax.net.ssl.keyStorePassword";
private static final String SYSTEM_PROPERTY_SSL_KEYSTORE_TYPE = "javax.net.ssl.keyStoreType";

private static final Pattern CERT_PATTERN = Pattern.compile(
        "-+BEGIN\\s+.*CERTIFICATE[^-]*-+(?:\\s|\\r|\\n)+" + "([a-z0-9+/=\\r\\n]+)" + "-+END\\s+.*CERTIFICATE[^-]*-+",
        Pattern.CASE_INSENSITIVE);

private static KeyStore exportX509CertToNewTempKeystore(String alias, String path, String password) {
    try {
        Container.ExecResult certificateExecResult = casContainer.execInContainer(String.format("keytool -exportcert -alias %s -keystore %s -rfc -storepass %s", alias, path, password).split(" "));
        KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());

        keyStore.load(null, null);
        Matcher matcher = CERT_PATTERN.matcher(certificateExecResult.toString());
        CertificateFactory certificateFactory = CertificateFactory.getInstance("X.509");
        List<X509Certificate> certificates = new ArrayList<>();

        int start = 0;
        while (matcher.find(start)) {
            byte[] buffer = Base64.getMimeDecoder().decode(matcher.group(1).getBytes(StandardCharsets.UTF_8));
            certificates.add((X509Certificate) certificateFactory.generateCertificate(new ByteArrayInputStream(buffer)));
            start = matcher.end();
        }

        for (X509Certificate certificate : certificates) {
            X500Principal subjectX500Principal = certificate.getSubjectX500Principal();
            keyStore.setCertificateEntry(subjectX500Principal.getName("RFC2253"), certificate);
        }

        return keyStore;
    } catch (IOException | KeyStoreException | NoSuchAlgorithmException | CertificateException | InterruptedException e) {
        throw new RuntimeException(e);
    }
}

private static String storeKeystoreToTempFile (KeyStore keyStore, String password, String prefix, String suffix) {
    try {
        Path tempTrustStore = Files.createTempFile(prefix, suffix);
        try (FileOutputStream fos = new FileOutputStream(tempTrustStore.toFile())) {
            keyStore.store(fos, password.toCharArray());
        }
        return tempTrustStore.toFile().getAbsolutePath();
    } catch (IOException | KeyStoreException | NoSuchAlgorithmException | CertificateException e) {
        throw new RuntimeException(e);
    }
}
```
- Configuration de `RestTemplate` permettant d'omettre la vérification des certificats lors d'appels HTTPS :
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class TestConfiguration {

    @Bean
    RestTemplateCustomizerSkipSslVerification getRestTemplateCustomizerSkipSslVerification() {
        return new RestTemplateCustomizerSkipSslVerification();
    }
}
```
```java
import org.springframework.boot.web.client.RestTemplateCustomizer;
import org.springframework.http.client.SimpleClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

import javax.net.ssl.*;
import java.io.IOException;
import java.net.HttpURLConnection;
import java.security.*;
import java.security.cert.X509Certificate;

public class RestTemplateCustomizerSkipSslVerification implements RestTemplateCustomizer {

    public void customize(RestTemplate restTemplate) {

        // copy of org.springframework.boot.actuate.autoconfigure.cloudfoundry.servlet.SkipSslVerificationHttpRequestFactory
        restTemplate.setRequestFactory(new SimpleClientHttpRequestFactory() {
            @Override
            protected void prepareConnection(HttpURLConnection connection, String httpMethod) throws IOException {
                if (connection instanceof HttpsURLConnection httpsURLConnection) {
                    prepareHttpsConnection((HttpsURLConnection) connection);
                }
                super.prepareConnection(connection, httpMethod);
            }
        });
    }

    private void prepareHttpsConnection(HttpsURLConnection connection) {
        connection.setHostnameVerifier(new SkipHostnameVerifier());
        try {
            SSLContext context = SSLContext.getInstance("TLS");
            context.init(null, new TrustManager[] { new SkipX509TrustManager() }, new SecureRandom());
            connection.setSSLSocketFactory(context.getSocketFactory());
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    private static class SkipHostnameVerifier implements HostnameVerifier {
        @Override
        public boolean verify(String s, SSLSession sslSession) {
            return true;
        }
    }

    private static class SkipX509TrustManager implements X509TrustManager {
        @Override
        public X509Certificate[] getAcceptedIssuers() {
            return new X509Certificate[0];
        }
        @Override
        public void checkClientTrusted(X509Certificate[] chain, String authType) {}
        @Override
        public void checkServerTrusted(X509Certificate[] chain, String authType) {}
    }
}
```
