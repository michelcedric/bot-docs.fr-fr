En règle générale, chaque message envoyé à l’utilisateur par un robot se rapporte directement à la saisie précédente de l’utilisateur.
Dans certains cas, il se peut qu’un bot ait besoin d’envoyer à l’utilisateur un message qui n’est pas directement associé au sujet de conversation actuel ou au dernier message envoyé par l’utilisateur. Il s’agit de **messages proactifs**.

Les messages proactifs peuvent être utiles dans différents cas de figure.
Lorsqu’un bot définit une minuterie ou un rappel, il peut avoir besoin d’avertir l’utilisateur lorsque le moment correspondant arrive.
Ou encore, lorsqu’un bot reçoit une notification émanant d’un système externe, il peut avoir besoin de communiquer immédiatement cette information à l’utilisateur.
Par exemple, si l’utilisateur a précédemment demandé au bot de surveiller le prix d’un produit, le bot peut alerter l’utilisateur dès que le prix du produit diminue de 20 %. En outre, si un bot a besoin de temps pour compiler une réponse à la question de l’utilisateur, il peut informer l’utilisateur de ce délai et autoriser la conversation à se poursuivre dans l’intervalle. Puis, dès que le bot a terminé de compiler la réponse à la question, il partage cette information avec l’utilisateur.

Lorsque vous implémentez des messages proactifs dans votre bot :

- *Évitez* d’envoyer plusieurs messages proactifs sur une courte période. Certains canaux appliquent des restrictions concernant la fréquence à laquelle un bot peut envoyer des messages à l’utilisateur, et désactivent le bot si ce dernier enfreint ces restrictions.
- *Évitez* d’envoyer des messages proactifs aux utilisateurs qui n’ont pas déjà interagi avec le bot ni sollicité de contact avec ce dernier par un autre moyen, tel qu’un e-mail ou un SMS.

Les messages proactifs peuvent produire un comportement inattendu. Considérons le scénario suivant.

![Manière dont les utilisateurs s’expriment](~/media/designing-bots/capabilities/proactive1.png)

Dans cet exemple, l’utilisateur a précédemment demandé au bot de surveiller les prix d’un hôtel à Las Vegas.
Le bot a donc lancé une tâche de surveillance en arrière-plan qui s’est exécutée en continu au cours des derniers jours.
Dans le cadre de cette conversation, l’utilisateur est en train de réserver un voyage à destination de Londres lorsque la tâche en arrière-plan déclenche un message de notification concernant l’existence d’une remise pour l’hôtel de Las Vegas. Le bot injecte cette information dans la conversation, ce qui déroute l’utilisateur.

Comment le bot aurait-il dû gérer cette situation ?

- Le bot aurait pu attendre que l’utilisateur ait terminé sa réservation actuelle, puis lui transmettre la notification. Cette approche se révèle la moins perturbatrice, mais le délai de communication de l’information concernant la remise risque de faire rater à l’utilisateur une opportunité intéressante pour l’hôtel de Las Vegas.
- Le bot aurait pu annuler le flux de réservation actuel et transmettre immédiatement la notification. Cette approche fournit les informations en temps voulu, mais risque de frustrer l’utilisateur en le contraignant à recommencer sa réservation depuis le début.
- Le bot aurait pu interrompre la réservation actuelle, basculer clairement le sujet de conversation vers l’hôtel de Las Vegas jusqu’à ce que l’utilisateur réponde sur ce sujet, puis revenir à la réservation en cours et reprendre à l’endroit exact où elle s’était interrompue. Cette approche constitue apparemment le meilleur choix, mais elle introduit une certaine complexité à la fois pour le développeur du bot et pour l’utilisateur.

Pour gérer les situations de ce type, votre bot utilisera généralement une combinaison de **messages proactifs ad hoc** et d’autres techniques.
