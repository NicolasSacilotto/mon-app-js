Pour ce tp : 

j'ai setup un environnement sur l'IDE Vscode qui contient le répertoire de mon conteneur docker dans lequel j'ai installé et configuré jenkins

tp node/jenkins, j'ai installé le plugin nodeJS

pour faire fonctionner la pipeline : 

Voici les étapes que j’ai suivies pour que le pipeline Jenkins Node.js fonctionne :

J’ai installé Node.js 18 et npm dans mon conteneur Ubuntu avec :
J’ai installé les dépendances du projet avec :
J’ai ajouté le reporter JUnit pour Jest dans mon package.json :
J’ai installé le reporter avec :
J’ai adapté mon Jenkinsfile pour que Jenkins récupère le rapport de test généré par Jest :
J’ai vérifié que le fichier test-results.xml est bien généré après les tests.