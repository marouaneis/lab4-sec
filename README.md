# lab4-sec
# Rapport d'Analyse Statique – UnCrackable-Level1 (OWASP MSTG)

**Date** : 14 mars 2026  
**Analyste** : Marouane  
**Localisation** : Marrakech, Maroc  

## Informations générales

- **APK analysé** : UnCrackable-Level1.apk  
- **Taille** : 66 Ko  
- **Version** : 1.0 (versionCode 1)  
- **Package** : owaspmstg.uncrackable1  
- **Provenance** : https://github.com/OWASP/mastg/raw/master/Crackmes/Android/Level_01/UnCrackable-Level1.apk  
- **Outils utilisés** :  
  - JADX-GUI (version récente 2025/2026)  
  - dex2jar v2.0  
  - PowerShell (extraction DEX, hash, listing ZIP)  
  - JD-GUI (vérification JAR optionnelle)




## Résumé exécutif

Cette analyse statique a révélé **4 vulnérabilités majeures** dans l'application UnCrackable-Level1, conçue comme un challenge éducatif OWASP.  

Les principales préoccupations sont :  
- **clé cryptographique entièrement hardcoded** (seed hex + ciphertext base64)  
- **mode AES-ECB insecure** combiné à une clé statique  
- **logs debug sensibles** exposant des informations crypto  
- **détections root/debug naïves** (facilement bypassables)

**Niveau de risque global** : **Élevé** (crypto cassée en statique pure – secret récupéré sans exécution)

**Actions prioritaires recommandées** :  
1. Supprimer toute clé / matériel sensible hardcoded (utiliser Android Keystore ou backend)  
2. Remplacer AES-ECB par AES-GCM (avec IV aléatoire et authentification)  
3. Supprimer les logs sensibles en build release (ProGuard + condition de log)

## Constats détaillés

### Constat #1 : Clé AES dérivée d'une seed hex hardcoded
**Sévérité** : Élevée  
**Description** : Une chaîne hex fixe sert de base à la génération de la clé AES 128 bits.  
**Localisation** : `sg.vantagepoint.uncrackable1.a` → `b("8d127684cbc37c17616d806cf50473cc")`  
**Impact potentiel** : Clé récupérable en clair → déchiffrement trivial du secret.  
**Remédiation** : Ne jamais stocker de clé dans le code. Utiliser KeyStore ou chiffrement dérivé du device/user.


### Constat #2 : Ciphertext du secret en base64 hardcoded
**Sévérité** : Élevée  
**Description** : Le secret chiffré est stocké en base64 directement dans le code.  
**Localisation** : `sg.vantagepoint.uncrackable1.a` → `Base64.decode("5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=", 0)`  
**Impact potentiel** : Secret ("I want to believe") récupéré statiquement en quelques secondes.  
**Remédiation** : Ne pas embarquer de données chiffrées avec clé présente dans l'APK.



### Constat #3 : Utilisation du mode AES-ECB (insecure)
**Sévérité** : Élevée  
**Description** : Mode ECB sans IV + PKCS7 + clé statique.  
**Localisation** : `sg.vantagepoint.a.a` → `Cipher.getInstance("AES/ECB/PKCS7Padding")`  
**Impact potentiel** : Mode cassé depuis 15+ ans (patterns répétitifs, attaques connues).  
**Remédiation** : Migrer vers AES-GCM ou CBC+HMAC.



### Constat #4 : Logs debug + détections root/debug naïves
**Sévérité** : Moyenne  
**Description** : Logs "CodeCheck" + "AES error:" + checks root/debug basiques.  
**Localisation** : `sg.vantagepoint.uncrackable1.a` (logs) + `MainActivity.onCreate()` (checks)  
**Impact potentiel** : Logs via Logcat + bypass facile (Frida, Smali patch).  
**Remédiation** : Retirer logs en release + renforcer anti-root (Play Integrity, multiple checks).


## Annexes

### Permissions demandées
Aucune permission dangereuse.  
Manifest minimal :  
- Pas de `INTERNET`, `WRITE_EXTERNAL_STORAGE`, `CAMERA`, etc.  
- Seulement permissions système de base (launcher activity).


### Composants exportés
- Activité principale : `sg.vantagepoint.uncrackable1.MainActivity` (exportée comme launcher)  
- Pas d'autres composants exportés sans protection.

### Autres éléments pertinents
- **Secret récupéré** : "I want to believe" (AES-ECB + clé hardcoded)  
- **Hash SHA-256 APK** : 1DA8BF7D266109F9A07C01BF7111A975CE01F190B9D914BCD3AE3D... (compléter avec ta valeur exacte)  
- **Structure APK** : classes.dex unique, pas de multi-dex.


**Fin du rapport**
