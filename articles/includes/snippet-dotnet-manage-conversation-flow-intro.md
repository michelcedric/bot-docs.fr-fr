Ce schéma illustre le flux à l’écran d’une application classique par rapport au flux de dialogue d’un bot. 

![bot](~/media/designing-bots/core/dialogs-screens.png)

Dans une application classique, tout commence par **l’écran principal**.
**L’écran principal** appelle **l’écran de nouvelle commande**.
**L’écran de nouvelle commande** conserve le contrôle jusqu’à ce qu’il se ferme ou appelle d’autres écrans. Si **l’écran de nouvelle commande** se ferme, l’utilisateur est renvoyé à **l’écran principal**.

Dans un bot, tout commence par le **dialogue racine**. Le **dialogue racine** appelle le **dialogue de nouvelle commande**. À ce stade, le **dialogue de nouvelle commande** prend le contrôle de la conversation et le conserve jusqu’à ce qu’il se ferme ou appelle d’autres dialogues. Si le **dialogue de nouvelle commande** se ferme, le contrôle de la conversation revient au **dialogue racine**.