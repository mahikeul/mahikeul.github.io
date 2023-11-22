# Java : fragments de code
## Autour des certificats
- Fortement inspiré de [&#x21aa; wsargent/Directions for creating PEM files](https://gist.github.com/wsargent/26c0d478d0901d4b0a11eeb10dcbe0d1)
```java
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
