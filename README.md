# 🔐 Certificate Extraction Flow - Visual Representation

## 🎨 **VISUAL FLOW DIAGRAM**

```
┌─────────────────────────────────────────────────────────────────────────────────────
│                           🚀 STARTING POINT                                        
│                                                                                     
│  📥 JKS Files (Base64 Encoded)                                                    
│  ┌─────────────────────┐                    ┌─────────────────────┐               
│  │ 🔒 TRUSTSTORE       │                    │ 🔑 KEYSTORE         │               
│  │    .JKS             │                    │     .JKS            │               
│  │                     │                    │                     │               
│  │ 📜 Root CA 1        │                    │ 👤 Client Cert      │               
│  │ 📜 Root CA 2        │                    │ 🔐 Private Key      │               
│  │ 📜 Root CA 3        │                    │                     │               
│  └─────────────────────┘                    └─────────────────────┘               
└─────────────────────────────────────────────────────────────────────────────────────
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────────
│                        ⚙️  ANSIBLE PROCESSING                                      
│                                                                                     
│  🗂️  Create: {{ workdir }}/tmp/temp_certs/                                      
│  🔓 Decode: Base64 → Binary JKS files                                            
│  🔍 Detect: Certificate aliases automatically                                    
└─────────────────────────────────────────────────────────────────────────────────────
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────────
│                        🔄 CERTIFICATE EXTRACTION                                   
│                                                                                     
│  ┌─────────────────────────────────────────────────────────────────────────────┐   
│  │                    🏛️  TRUSTSTORE PROCESSING                                │   
│  │                                                                             │   
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                    │   
│  │  │ 📜 Root CA1 │    │ 📜 Root CA2 │    │ 📜 Root CA3 │                    │   
│  │  │             │    │             │    │             │                    │   
│  │  │ 🔧 keytool  │    │ 🔧 keytool  │    │ 🔧 keytool  │                    │   
│  │  │ -export     │    │ -export     │    │ -export     │                    │   
│  │  │ -rfc        │    │ -rfc        │    │ -rfc        │                    │   
│  │  └─────────────┘    └─────────────┘    └─────────────┘                    │   
│  │           │                       │                       │                │   
│  │           ▼                       ▼                       ▼                │   
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                    │   
│  │  │ 📄 ca_cert1 │    │ 📄 ca_cert2 │    │ 📄 ca_cert3 │                    │   
│  │  │   .pem      │    │   .pem      │    │   .pem      │                    │   
│  │  └─────────────┘    └─────────────┘    └─────────────┘                    │   
│  └─────────────────────────────────────────────────────────────────────────────┘   
│                                                                                     
│  ┌─────────────────────────────────────────────────────────────────────────────┐   
│  │                    🔑 KEYSTORE PROCESSING                                    │   
│  │                                                                             │   
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                    │   
│  │  │ 🔄 JKS→PKCS│    │ 📜 Extract  │    │ 🔐 Extract  │                    │   
│  │  │   12        │    │   Client    │    │  Private    │                    │   
│  │  │             │    │   Cert      │    │    Key      │                    │   
│  │  │ 🔧 keytool  │    │ 🔧 keytool  │    │ 🔧 openssl  │                    │   
│  │  │ -import     │    │ -export     │    │  pkcs12     │                    │   
│  │  │ -keystore   │    │ -rfc        │    │ -nocerts    │                    │   
│  │  └─────────────┘    └─────────────┘    └─────────────┘                    │   
│  │           │                       │                       │                │   
│  │           ▼                       ▼                       ▼                │   
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                    │   
│  │  │ 📦 keystore │    │ 📄 client   │    │ 🔐 client  │                    │   
│  │  │   .p12      │    │   _cert.pem │    │   _key.pem  │                    │   
│  │  └─────────────┘    └─────────────┘    └─────────────┘                    │   
│  └─────────────────────────────────────────────────────────────────────────────┘   
└─────────────────────────────────────────────────────────────────────────────────────
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────────
│                        🎯 KUBERNETES SECRET                                        
│                                                                                     
│  🔐 Secret Name: kafka-soct-certs                                                  
│  📍 Namespace: observability                                                       
│                                                                                     
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           
│  │ 📄 ca_cert1 │  │ 📄 ca_cert2 │  │ 📄 client  │  │ 🔐 client  │           
│  │   .pem      │  │   .pem      │  │   _cert    │  │   _key     │           
│  │ (Base64)    │  │ (Base64)    │  │   .pem     │  │   .pem     │           
│  │             │  │             │  │ (Base64)   │  │ (Base64)   │           
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘           
│                                                                                     
│  ┌─────────────┐                                                                   
│  │ 🔑 client   │                                                                   
│  │   _key_pass│                                                                   
│  │ (Base64)   │                                                                   
│  └─────────────┘                                                                   
└─────────────────────────────────────────────────────────────────────────────────────
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────────
│                        🧹 CLEANUP & OUTPUT                                         
│                                                                                     
│  🗑️  Remove: {{ workdir }}/tmp/temp_certs/                                       
│  ✅ Secret: Ready for Fluentd configuration                                       
│  🔒 SSL/TLS: Secure Kafka connection enabled                                      
└─────────────────────────────────────────────────────────────────────────────────────
```

## 🔧 **COMMAND FLOW VISUAL**

```
┌─────────────────────────────────────────────────────────────────────────────────────
│                        🔧 COMMAND EXECUTION FLOW                                   
│                                                                                     
│  1️⃣  CREATE TEMP DIRECTORY                                                        
│      ┌─────────────────────────────────────────────────────────────────────────┐   
│      │ mkdir -p {{ workdir }}/tmp/temp_certs/                                 │   
│      └─────────────────────────────────────────────────────────────────────────┘   
│                                                                                     
│  2️⃣  DECODE JKS FILES                                                             
│      ┌─────────────────────────────────────────────────────────────────────────┐   
│      │ echo "{{ values_nemoadapter_ossit_oceankafka_truststore }}" | base64 -d │   
│      │ echo "{{ values_nemoadapter_ossit_oceankafka_keystore }}" | base64 -d   │   
│      └─────────────────────────────────────────────────────────────────────────┘   
│                                                                                     
│  3️⃣  LIST ALIASES                                                                 
│      ┌─────────────────────────────────────────────────────────────────────────┐   
│      │ keytool -list -keystore truststore.jks -storepass <password>           │   
│      │ keytool -list -keystore keystore.jks -storepass <password>             │   
│      └─────────────────────────────────────────────────────────────────────────┘   
│                                                                                     
│  4️⃣  EXTRACT CERTIFICATES                                                          
│      ┌─────────────────────────────────────────────────────────────────────────┐   
│      │ keytool -exportcert -alias <alias> -file ca_cert_1.pem -rfc            │   
│      │ keytool -exportcert -alias <alias> -file ca_cert_2.pem -rfc            │   
│      │ keytool -exportcert -alias 1 -file client_cert.pem -rfc                │   
│      └─────────────────────────────────────────────────────────────────────────┘   
│                                                                                     
│  5️⃣  CONVERT & EXTRACT PRIVATE KEY                                                
│      ┌─────────────────────────────────────────────────────────────────────────┐   
│      │ keytool -importkeystore -srckeystore keystore.jks -destkeystore keystore.p12 │   
│      │ openssl pkcs12 -in keystore.p12 -out client_cert_key.pem -nocerts      │   
│      └─────────────────────────────────────────────────────────────────────────┘   
│                                                                                     
│  6️⃣  CREATE KUBERNETES SECRET                                                    
│      ┌─────────────────────────────────────────────────────────────────────────┐   
│      │ kubectl create secret generic kafka-soct-certs --from-file=ca_cert_1.pem │   
│      └─────────────────────────────────────────────────────────────────────────┘   
│                                                                                     
│  7️⃣  CLEANUP                                                                      
│      ┌─────────────────────────────────────────────────────────────────────────┐   
│      │ rm -rf {{ workdir }}/tmp/temp_certs/                                   │   
│      └─────────────────────────────────────────────────────────────────────────┘   
└─────────────────────────────────────────────────────────────────────────────────────
```

## 📊 **DATA TRANSFORMATION VISUAL**

```
┌─────────────────────────────────────────────────────────────────────────────────────
│                        📊 DATA TRANSFORMATION FLOW                                 
│                                                                                     
│  INPUT FORMAT                    PROCESSING                    OUTPUT FORMAT        
│  ───────────                     ───────────                    ───────────         
│                                                                                     
│  🔒 TRUSTSTORE.JKS               🔄 keytool -export            📄 ca_cert_1.pem     
│  (Base64 String)                 (JKS → PEM)                   (PEM Certificate)   
│  ┌─────────────────┐             ┌─────────────────┐          ┌─────────────────┐ 
│  │ 📜 Root CA 1    │ ──────────▶ │ 🔧 keytool      │ ───────▶ │ 📄 ca_cert_1    │ 
│  │ 📜 Root CA 2    │             │ -export -rfc    │          │   .pem          │ 
│  │ 📜 Root CA 3    │             └─────────────────┘          └─────────────────┘ 
│  └─────────────────┘                                             📄 ca_cert_2.pem 
│                                                                  📄 ca_cert_3.pem 
│                                                                                     
│  🔑 KEYSTORE.JKS                  🔄 JKS → PKCS12 → PEM        📄 client_cert.pem  
│  (Base64 String)                  (Multi-step conversion)      (PEM Certificate)   
│  ┌─────────────────┐             ┌─────────────────┐          ┌─────────────────┐ 
│  │ 👤 Client Cert  │ ──────────▶ │ 🔧 keytool      │ ───────▶ │ 📄 client_cert  │ 
│  │ 🔐 Private Key  │             │ -importkeystore │          │   .pem          │ 
│  └─────────────────┘             │ 🔧 openssl      │          │ 🔐 client_cert  │ 
│                                   │ pkcs12         │          │   _key.pem      │ 
│                                   └─────────────────┘          └─────────────────┘ 
└─────────────────────────────────────────────────────────────────────────────────────
```

## 🎯 **FINAL OUTPUT STRUCTURE**

```
┌─────────────────────────────────────────────────────────────────────────────────────
│                        🎯 KUBERNETES SECRET STRUCTURE                              
│                                                                                     
│  🔐 kafka-soct-certs (observability namespace)                                    
│  ┌─────────────────────────────────────────────────────────────────────────────┐   
│  │                                                                             │   
│  │  📄 ca_cert_1.pem: "LS0tLS1CRUdJTi..." (Base64 encoded)                   │   
│  │  📄 ca_cert_2.pem: "LS0tLS1CRUdJTi..." (Base64 encoded)                   │   
│  │  📄 client_cert.pem: "LS0tLS1CRUdJTi..." (Base64 encoded)                 │   
│  │  🔐 client_cert_key.pem: "LS0tLS1CRUdJTi..." (Base64 encoded)             │   
│  │  🔑 client_cert_key_password: "cGFzc3dvcmQ=" (Base64 encoded)             │   
│  │                                                                             │   
│  └─────────────────────────────────────────────────────────────────────────────┘   
│                                                                                     
│  ✅ READY FOR FLUENTD CONFIGURATION                                               
│  ✅ ENABLES SSL/TLS CONNECTION TO EXTERNAL KAFKA                                 
│  ✅ AUTOMATIC JKS TO PEM CONVERSION                                               
└─────────────────────────────────────────────────────────────────────────────────────
```

## 🚀 **WHAT THIS ENABLES**

```
┌─────────────────────────────────────────────────────────────────────────────────────
│                        🚀 SYSTEM BENEFITS                                          
│                                                                                     
│  🔒 SECURITY                    🔄 AUTOMATION                    📊 MONITORING     
│  ─────────                      ───────────                      ─────────         
│                                                                                     
│  • SSL/TLS encryption          • No manual certificate          • Centralized      
│  • Secure Kafka connection     • conversion needed              • logging          
│  • Certificate validation      • Automatic alias detection      • Observability    
│  • Private key protection      • Dynamic CA handling            • Fluentd → Kafka  
│                                                                                     
│  🎯 DEPLOYMENT                 🧹 MAINTENANCE                   🔧 FLEXIBILITY     
│  ───────────                   ───────────                      ─────────         
│                                                                                     
│  • Kubernetes secret ready     • Temporary file cleanup         • Multiple CA      
│  • One-command deployment      • No leftover files              • support          
│  • Consistent configuration    • Secure processing              • JKS format       
│  • Environment agnostic        • Password protection            • input support    
└─────────────────────────────────────────────────────────────────────────────────────
```
