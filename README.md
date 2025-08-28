Pour ce tp : 

j'ai setup un environnement sur l'IDE Vscode qui contient le répertoire de mon conteneur docker dans lequel j'ai installé et configuré jenkins

tp node/jenkins, j'ai installé le plugin nodeJS

pour faire fonctionner la pipeline : 

Exercice 1 : Premier déploiement
Pour ce TP, j’ai configuré un environnement sur l’IDE VS Code, qui pointe vers le répertoire de mon conteneur Docker où Jenkins est installé et configuré. J’ai également installé le plugin NodeJS pour Jenkins.

Pour faire fonctionner la pipeline Node.js avec Jenkins, j’ai procédé ainsi :
J’ai installé Node.js 18 et npm dans le conteneur Ubuntu. Ensuite, j’ai installé les dépendances du projet avec la commande appropriée. J’ai ajouté le reporter JUnit pour Jest dans le fichier package.json, puis j’ai installé ce reporter. J’ai modifié le Jenkinsfile afin que Jenkins puisse récupérer le rapport de test généré par Jest. Enfin, j’ai vérifié que le fichier test-results.xml est bien créé après l’exécution des tests.

2. Exercice 2 : Gestion des branches
Création d'une branche develop, création d'un projet multi branch qui permet de récupérer toutes les branches du repo git et de pouvoir de déclencher la pipeline de la branche main ou la branche develop 


3. Exercice 3 : Tests et qualité
test rajouté : 
test('Tests échoué', () => {
        expect(isValidNumber(Infinity)).toBe(true);
});