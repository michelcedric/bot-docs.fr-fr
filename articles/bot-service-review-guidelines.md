# <a name="bot-review-guidelines"></a>Instructions d’évaluation des bots

Nous vous souhaitons la bienvenue et vous remercions d’investir vos talents et votre temps dans la création de robots, de botlets, d’applications web, de compléments ou de compétences (« intégrations d’applications ») pour les canaux Microsoft. Vous trouverez ci-dessous les conditions minimales requises pour que votre intégration d’application puisse être publiée sur un canal Microsoft tel que Skype ou Microsoft Teams. Chaque canal peut imposer des conditions spécifiques en plus de celles détaillées ci-dessous. Le cas échéant, vous trouverez les conditions spécifiques de chaque canal sur la page de configuration de celui-ci, et devrez peut-être vous inscrire au service d’un canal avant de pouvoir publier un robot sur celui-ci.

## <a name="app-integration-policies"></a>Stratégies d’intégration d’applications
###  <a name="1-value-representation-security-and-usability"></a>1. Valeur, représentation, sécurité et convivialité.

L’intégration de votre application et les métadonnées qui y sont associées doivent présenter les caractéristiques suivantes :

- avoir des métadonnées distinctes et informatives, et offrir une expérience utilisateur de valeur et de qualité ;
- refléter de manière précise et claire la source, la fonctionnalité et les fonctions de votre intégration d’application, ainsi que décrire les limitations importantes ;
- ne pas utiliser un nom ou une icône similaire à ceux d’autres applications, et ne pas prétendre représenter une société, d’un organe officiel ou toute autre entité sans autorisation formelle ;
- ne pas menacer ou compromettre la sécurité des utilisateurs, ou la sécurité ou la fonctionnalité du canal Microsoft ;
- ne pas tenter de modifier ou d’étendre la fonctionnalité décrite en violation de ces politiques ou des conditions applicables au canal Microsoft ;
- ne pas inclure ou activer des programme malveillant ;
- être testable ;
- fonctionner sans discontinuité et rester réactive aux entrées des utilisateurs ; 
- inclure un lien fonctionnel vers vos conditions d’utilisation ;
- opérer conformément à la description, au profil, aux conditions d’utilisation et à la politique de confidentialité exposées ;
- opérer conformément aux conditions applicables au Microsoft Bot Framework (ou à d’autres conditions applicables à son développement), ainsi qu’aux conditions applicables à Microsoft Channel (ou à d’autres conditions applicables au canal sur lequel votre intégration d’applications est publiée) ;
- informer les utilisateurs si l’intégration d’application inclut une interaction humaine (par exemple, service ou support clientèle opéré par une personne en direct) ;
- être localisée dans tous les langues prises en charge. Le texte de description de votre intégration d’application doit être localisé dans toutes les langues que vous déclarez.

### <a name="2--privacy"></a>2.  Confidentialité

- Si votre intégration de votre application gère des informations personnelles des utilisateurs (toutes informations ou données identifiant ou utilisables pour identifier une personne, ou associées à de telles informations ou données). Exemples d’informations personnelles : nom et adresse, numéro de téléphone, identificateurs biométriques, emplacement, contacts, photos, enregistrements audio et vidéo, documents, SMS, e-mail ou autre communication textuelle, captures d’écran et, dans certains cas, historique de navigation combiné. Vous devez fournir un lien bien visible entre votre intégration d’application et une politique de confidentialité applicable, laquelle doit être conforme à l’ensemble des lois, réglementations et exigences en matière de politique applicables. Cette politique doit couvrir les données que vous collectez ou transmettez, ce que vous allez en faire et, les personnes avec lesquelles vous allez les partager. Si vous ne disposez pas d’une déclaration de confidentialité, voici quelques ressources tierces * qui pourraient vous aider :

Future of Privacy Forum : [Générateur de politique de confidentialité d’application](http://www.applicationprivacy.org/do-tools/privacy-policy-generator/)

Iubenda : [Générateur de politique de confidentialité](http://www.iubenda.com/en)

*_Vous acceptez d’assumer l’ensemble des risques et responsabilités découlant de votre utilisation de ces ressources tierces, et reconnaissez que Microsoft n’est pas responsable d’éventuels problèmes résultant de votre utilisation de ces ressources._
- Votre intégration d’application ne doit pas collecter, stocker ou transmettre des informations personnelles sans rapport avec son objectif principal, sans l’obtention préalable du consentement exprès de l’utilisateur. Vous devez obtenir le consentement des utilisateurs au traitement des informations personnelles lorsque la loi l’exige. 
- Vous ne pouvez pas publier une intégration d’application destinée à des enfants de moins de 13 ans (comme décrit dans la loi COPPA -Children’s Online Privacy Protection Act- aux États-Unis), sans l’autorisation expresse de Microsoft.

### <a name="3--financial-transactions"></a>3.  Transactions financières
- Pour les intégrations d’application avec paiement activé : 
  - Votre intégration d’application peut ne pas transmettre de détails de l’instrument financier via l’interface utilisateur.
  - Sous réserve de toute restriction de canal ou de plateforme tierce, votre intégration d’application peut : (a) prendre en charge les paiements via le [Centre des vendeurs Microsoft](https://seller.microsoft.com/) conformément aux conditions contractuelles dudit centre ; ou (b) transmettre des liens vers d’autres services de paiement sécurisé ;
  - si votre intégration d’application active les mécanismes de paiement susmentionnés, vous devez le spécifier dans les conditions d’utilisation et la politique de confidentialité de votre application (ainsi que dans toute page de profil ou site web concernant l’intégration d’application) avant que l’utilisateur final accepte d’utiliser votre intégration d’application ;
  - Vous devez indiquer clairement dans le profil d’une intégration d’application si celle-ci d’application fait l’objet d’un paiement, et fournir aux utilisateurs finaux les détails du service client du marchand ;
  - Vous ne pouvez pas publier sur un canal Microsoft des intégrations d’application incluant des liens ou dirigeant les utilisateurs vers des services de paiement pour l’achat de produits numériques sans l’autorisation expresse de Microsoft.

### <a name="4--content"></a>4.  Contenu 
- Tout le contenu de votre intégration d’application et des métadonnées associées, doit être créé à l’origine par l’éditeur, dûment autorisé par le détenteur de droits tiers, utilisé dans la mesure autorisée par celui-ci, ou utilisé conformément à ce que la loi autorise.
- Vous vous interdisez de publier une intégration d’application ou du contenu dans votre intégration d’application présentant les caractéristiques suivantes : 
  - est illégal ;
  - abuse les enfants, leur nuit ou risque leur de nuire ;
  - inclut de la publicité, du courrier indésirable, des communications, publications ou messages en bloc indésirables ou non sollicités ;
  - pourrait être considéré comme inapproprié ou offensant (par exemple, nudité, bestialité, blasphème, pornographie ou sexualité explicite, violence graphique ou gratuite, tabagie, activités criminelles, dangereuses ou irresponsables, drogue, violations des droits humains, création d’armes ou utilisation illégale de celles-ci contre des personnes ou animaux du monde réel, utilisation irresponsable de produits alcoolisés) ;
  - est faux ou trompeur (par exemple, demande l’argent sous de faux prétextes, usurpation d’identité) ;
  - est préjudiciable pour vous, Microsoft Channel ou des tiers, ou crée un risque pour la sécurité (par exemple, transmission de virus, harcèlement, publication de contenu terroriste, tenue de propos incitant à la haine, apologie de la discrimination, de la haine ou de la violence sur la base de considérations de race, d’appartenance ethnique, d’origine nationale, de langue, de genre, d’âge, de handicap, de religion, d’orientation sexuelle, de statut de vétéran ou d’appartenance à tout autre groupe social) ;
  - porte atteinte aux droits d’autrui (par exemple, partage non autorisé de musique ou de tout autre contenu protégés par droits d’auteur) ;
  - présente un caractère diffamatoire, pamphlétaire, calomnieux ou menaçant ;
  - viole la vie privée de tiers ; 
  - traite des informations qui (a) ont trait à l’état, au traitement ou au paiement du traitement d’un patient ; (b) identifie des personnes bénéficiant couverture santé ou communique avec des patients, des membres ou des bénéficiaires de couverture santé ; ou (c) constitue une information médicale protégée en vertu de la loi Health Insurance Portability and Accountability Act (HIPAA) telle qu’amendée, ou ayant trait à toute activité régie par l’HIPAA si vous êtes une « entité couverte » ou un « partenaire commercial » aux termes de l’HIPAA ;
  - présente un caractère offensant dans un pays ou une région ciblés par l’application, un contenu pouvant être considéré comme offensant dans certains pays ou régions en raison de lois ou de normes culturelles locales ;
  - permet à d’autres personnes d’enfreindre ces règles. 
- Si vous voulez apporter des modifications matérielles à votre intégration d’application, vous devez avertir Microsoft au préalable en envoyant un e-mail au canal sur lequel vous l’avez publiée.  Les modifications apportées à l’inscription de votre robot peuvent nécessiter un réexamen de celui-ci pour s’assurer qu’il continue à répondre aux exigences énoncées ici.  Microsoft se réserve le droit, à sa seule discrétion, d’examiner de façon intermittente les intégrations d’application sur tout canal, et, le cas échéant, de les supprimer sans préavis.
