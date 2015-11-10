---
layout: post
title: MediaWiki e Single-Sign-On in Active Directory
---

Per vari motivi abbiamo deciso di avere un wiki interno all'azienda, dove annotare le varie procedure interne, e piu' in generale 
da usare come strumento collaborativo.
Dopo averne valutati alcuni, abbiamo scelto MediaWiki, e l'abbiamo quindi installato su una VM con Linux OpenSuse 13.1.
Non descriverò qui la procedura di installazione, visto che è piuttosto semplice.
Ciò che volevamo ottenere era un ambiente dove gli utenti potessero contribuire articoli nel modo più semplice possibile.
Una delle cose che gli utenti trovano più fastidiose, quando devono iniziare ad utilizzare un nuovo strumento/ambiente, in base alla mia esperienza, è il
fatto di dover ricordare/segnarsi/creare una nuova password. Per evitare questo problema, abbiamo deciso di implementare un meccanismo di Single-Sign-On,
integrato con Active Directory, il nostro ambiente di autenticazione aziendale.
MediaWiki ha già un'estensione che può interfacciarsi con LDAP (quindi anche Active Directory), e può quindi essere usata per raccogliere le varie
informazioni relative all'utente loggato, però per evitare di fargli inserire la password, l'autenticazione deve avvenire via Kerberos a livello del
web server (Apache, nel nostro caso). 
L'implementazione di tale meccanismo richiede quindi di mettere mano sia alla configurazione di MediaWiki che di Apache.

Autenticazione Kerberos in Apache

Per poter funzionare, si deve creare un SPN (Service Principal Name) per il servizio HTTP sull'host, e aggiungere la keytab relativa.
Per farlo ci sono vari modi. A mio parere il piu' semplice, se il server è già in dominio, è il seguente:

Da shell:
{{{
group:> net ads keytab add HTTP -U <admin>
}}}

dove <admin> è un utente con diritti di Domain Admin in Active Directory.
Questo comando crea sia il SPN in AD, sia aggiunge la keytab al sistema.

Aggiungere le righe seguenti alla configurazione della directory MediaWiki in Apache:

{{{
    AuthType Kerberos
    AuthName "MediaWiki Authentication"
    KrbMethodNegotiate On
    KrbMethodK5Passwd On
    KrbVerifyKDC on
    KrbAuthRealm NomeDominio
    Krb5Keytab /etc/apache2/httpd.keytab
    KrbLocalUserMapping On
    require valid-user

}}}







