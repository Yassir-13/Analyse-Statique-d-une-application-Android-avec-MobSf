# Analyse-Statique-d-une-application-Android-avec-MobSf

* **Date d'analyse :** 2026-03-11
* **Analyste :** Yassir Nacir
* **APK analysé :** vulnbank3.apk (SHA-256: b9f87d81498671560e6c083a354aac35fd8407d0f63457f71a9b37b04ee97a827)
* **Version :** Non disponible
* **Outils utilisés :** MobSF (version non disponible) dans VM Mobexler v4.4.5

---

## Résumé exécutif

L’analyse de l’application vulnbank3.apk révèle plusieurs vulnérabilités critiques affectant la sécurité des données, des communications et du code.
Des faiblesses majeures incluent l’utilisation de trafic non chiffré, la présence de secrets en dur dans le code et l’exécution de requêtes SQL non sécurisées.
L’application est également signée avec un certificat de debug et utilise des mécanismes cryptographiques faibles (SHA1).
Le score de sécurité global est faible (37/100), indiquant un niveau de risque élevé.

---

## Vulnérabilités critiques

### 1. Application signée avec un certificat de debug

* **Sévérité :** Élevée
* **Catégorie MASVS :** MSTG-RESILIENCE-1 / MSTG-CODE-1
* **Description :** L’application est signée avec un certificat de debug au lieu d’un certificat de production.
* **Preuve :** Section "Certificate Analysis" → Debug certificate détecté
* **Impact :** Permet la modification, la re-signature et facilite le reverse engineering
* **Remédiation :** Signer l’application avec un certificat de production sécurisé

---

### 2. Utilisation d’un algorithme cryptographique faible (SHA1)

* **Sévérité :** Élevée
* **Catégorie MASVS :** MSTG-CRYPTO-1
* **Description :** L’application utilise SHA1withRSA, vulnérable aux collisions
* **Preuve :** Certificate Analysis → SHA1withRSA
* **Impact :** Risque de falsification de signature
* **Remédiation :** Utiliser SHA-256 ou supérieur

---

### 3. Communication en clair autorisée

* **Sévérité :** Élevée
* **Catégorie MASVS :** MSTG-NETWORK-1
* **Description :** L’application autorise le trafic HTTP non chiffré
* **Preuve :** android:usesCleartextTraffic=true + config réseau absent
* **Impact :** Attaques Man-in-the-Middle, interception et modification des données
* **Remédiation :** Forcer HTTPS et implémenter un Network Security Config

---

### 4. Injection SQL possible

* **Sévérité :** Élevée
* **Catégorie MASVS :** MSTG-CODE-8
* **Description :** Exécution de requêtes SQL brutes avec entrées utilisateur non filtrées
* **Preuve :**

  * AsyncLocalStorageUtil.java
  * ReactDatabaseSupplier.java
* **Impact :** Accès non autorisé aux données
* **Remédiation :** Utiliser des requêtes paramétrées (Prepared Statements)

---

### 5. Secrets sensibles stockés en dur

* **Sévérité :** Élevée
* **Catégorie MASVS :** MSTG-STORAGE-14
* **Description :** Présence de clés et données sensibles dans le code source
* **Preuve :**

  * Secrets.java
  * Section "Possible Hardcoded Secrets"
* **Impact :** Compromission totale de l’application
* **Remédiation :** Utiliser un stockage sécurisé (Keystore, backend sécurisé)

---

## Autres observations

* Application installable sur Android 7.0 (systèmes vulnérables) → MSTG-PLATFORM-1
* Backup activé (android:allowBackup=true) → MSTG-STORAGE-1
* Broadcast Receiver exporté → MSTG-PLATFORM-2
* Données sensibles potentiellement loggées → MSTG-STORAGE-3
* Endpoint de debug exposé : http://192.168.18.5:5000/debug/users
* Absence de configuration réseau sécurisée

---

## Recommandations prioritaires

1. Désactiver le trafic en clair et imposer HTTPS
2. Supprimer tous les secrets du code source
3. Corriger les requêtes SQL (requêtes préparées)
4. Signer l’application avec un certificat de production sécurisé
5. Désactiver allowBackup et sécuriser les composants exportés

---

## Annexes

### Permissions

* android.permission.INTERNET
* com.vulnerablebankapp.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION

### Composants exportés

* com.vulnerablebankapp.MainActivity
* android.permission.DUMP

### Endpoints identifiés

* http://192.168.18.5:5000/debug/users
* https://reactnative.dev/docs/debugging

---
