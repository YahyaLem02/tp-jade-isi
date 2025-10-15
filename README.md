# TP JADE : Multi-Agents avec cycle de vie - Version Complète et Commentée

**Préparé par :** EL AOUMARI Abdelmoughith & LEMKHARBECH Yahya  
**Matière :** Intelligence Artificielle Distribuée - Ingénierie des Connaissances

---

## 📋 ÉNONCÉ DU TP

### Objectifs pédagogiques
- Comprendre le cycle de vie des agents JADE
- Implémenter la communication entre agents
- Maîtriser les différents types de comportements (Behaviours)
- Créer un système multi-agents simple mais fonctionnel

### Contexte du TP
Ce TP consiste à créer un système de vente en ligne simplifié avec trois types d'agents :
1. **Agent Vendeur** : Répond aux demandes de produits
2. **Agent Acheteur** : Effectue des demandes d'achat
3. **Agent Observateur** : Surveille le système périodiquement

### Compétences développées
- Configuration et démarrage de JADE
- Création d'agents avec différents comportements
- Communication par messages ACL (Agent Communication Language)
- Gestion du cycle de vie des agents

---

## 🏗️ Structure du projet

```
projet-jade/
├── src/
│   └── agents/
│       ├── AgentVendeur.java      # Agent qui vend des produits
│       ├── AgentAcheteur.java     # Agent qui achète des produits
│       ├── AgentObservateur.java  # Agent qui observe le système
│       └── Main.java              # Point d'entrée du programme
└── lib/
    └── jade.jar                   # Bibliothèque JADE
```

---

## 💻 Code Source Commenté

### 1. AgentVendeur.java

```java
package agents;

import jade.core.Agent;                    // Classe de base pour tous les agents JADE
import jade.core.AID;                     // Agent Identifier - identifiant unique d'un agent
import jade.core.behaviours.CyclicBehaviour; // Comportement qui s'exécute en boucle infinie
import jade.lang.acl.ACLMessage;          // Message de communication entre agents

/**
 * AGENT VENDEUR
 * Rôle : Recevoir les demandes d'achat et répondre avec la disponibilité des produits
 * Comportement : Écoute en permanence les messages entrants (CyclicBehaviour)
 */
public class AgentVendeur extends Agent {

    /**
     * Méthode setup() : Premier événement du cycle de vie de l'agent
     * Appelée automatiquement lors de la création de l'agent
     * Utilité : Initialiser l'agent, ajouter des comportements, configurer les ressources
     */
    @Override
    protected void setup() {
        System.out.println(getLocalName() + " : setup() -> Initialisation du vendeur.");
        
        // Ajout d'un comportement cyclique pour traiter les messages
        // CyclicBehaviour = comportement qui se répète indéfiniment
        addBehaviour(new CyclicBehaviour() {
            
            /**
             * Méthode action() : Cœur du comportement de l'agent
             * Appelée en boucle tant que l'agent est actif
             */
            @Override
            public void action() {
                // receive() : Récupère un message de la boîte aux lettres de l'agent
                ACLMessage msg = receive();
                
                if (msg != null) {
                    // Message reçu : le traiter
                    System.out.println(getLocalName() + " a reçu : " + msg.getContent());
                    
                    // Créer une réponse basée sur le message reçu
                    // createReply() : Crée automatiquement un message de réponse avec les bons destinataires
                    ACLMessage reply = msg.createReply();
                    reply.setPerformative(ACLMessage.INFORM); // Type de message : INFORM (information)
                    reply.setContent("Réponse du vendeur : Produit disponible !");
                    
                    // Envoyer la réponse
                    send(reply);
                    System.out.println(getLocalName() + " a envoyé une réponse.");
                } else {
                    // Aucun message : bloquer le comportement jusqu'à réception d'un message
                    // block() : Met en pause le comportement pour économiser les ressources
                    block();
                }
            }
        });
    }

    /**
     * Méthode takeDown() : Dernière méthode appelée avant la destruction de l'agent
     * Utilité : Nettoyer les ressources, sauvegarder des données, fermer des connexions
     */
    @Override
    protected void takeDown() {
        System.out.println(getLocalName() + " : takeDown() -> Terminaison de l'agent vendeur.");
    }
}
```

### 2. AgentAcheteur.java

```java
package agents;

import jade.core.Agent;
import jade.core.AID;
import jade.core.behaviours.OneShotBehaviour; // Comportement qui s'exécute une seule fois
import jade.core.behaviours.CyclicBehaviour;
import jade.lang.acl.ACLMessage;

/**
 * AGENT ACHETEUR
 * Rôle : Envoyer des demandes d'achat et traiter les réponses
 * Comportements : 
 * - OneShotBehaviour pour envoyer une demande
 * - CyclicBehaviour pour écouter les réponses
 */
public class AgentAcheteur extends Agent {

    @Override
    protected void setup() {
        System.out.println(getLocalName() + " : setup() -> Initialisation de l'acheteur.");
        
        // COMPORTEMENT 1 : Envoyer une demande (une seule fois)
        // OneShotBehaviour = comportement qui s'exécute une fois puis se termine
        addBehaviour(new OneShotBehaviour() {
            @Override
            public void action() {
                // Attendre un peu pour s'assurer que le vendeur est initialisé
                try {
                    Thread.sleep(1000); // Pause de 1 seconde
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                
                // Créer un nouveau message de type REQUEST (demande)
                ACLMessage msg = new ACLMessage(ACLMessage.REQUEST);
                
                // Ajouter le destinataire : l'agent vendeur
                // AID.ISLOCALNAME = l'agent est dans le même conteneur
                msg.addReceiver(new AID("vendeur", AID.ISLOCALNAME));
                
                // Définir le contenu du message
                msg.setContent("Bonjour, je veux acheter un produit.");
                
                // Envoyer le message
                send(msg);
                System.out.println(getLocalName() + " a envoyé une demande au vendeur.");
            }
        });
        
        // COMPORTEMENT 2 : Écouter les réponses (en continu)
        addBehaviour(new CyclicBehaviour() {
            @Override
            public void action() {
                ACLMessage msg = receive();
                if (msg != null) {
                    System.out.println(getLocalName() + " a reçu la réponse : " + msg.getContent());
                } else {
                    block(); // Attendre un message
                }
            }
        });
    }

    @Override
    protected void takeDown() {
        System.out.println(getLocalName() + " : takeDown() -> Terminaison de l'agent acheteur.");
    }
}
```

### 3. AgentObservateur.java

```java
package agents;

import jade.core.Agent;
import jade.core.behaviours.TickerBehaviour; // Comportement qui s'exécute périodiquement

/**
 * AGENT OBSERVATEUR
 * Rôle : Surveiller le système et afficher des informations périodiquement
 * Comportement : TickerBehaviour pour des actions répétitives avec intervalle fixe
 */
public class AgentObservateur extends Agent {

    @Override
    protected void setup() {
        System.out.println(getLocalName() + " : setup() -> Initialisation de l'observateur.");
        
        // TickerBehaviour : Comportement qui s'exécute à intervalles réguliers
        // Paramètres : (agent, période_en_millisecondes)
        addBehaviour(new TickerBehaviour(this, 3000) { // Toutes les 3 secondes
            
            /**
             * Méthode onTick() : Appelée à chaque intervalle défini
             * Utilité : Effectuer des tâches de surveillance, de monitoring, etc.
             */
            @Override
            protected void onTick() {
                System.out.println(getLocalName() + " : Observation du système à " + 
                    new java.util.Date());
                
                // Ici, on pourrait ajouter :
                // - Vérification de l'état des autres agents
                // - Collecte de statistiques
                // - Surveillance de la performance du système
            }
        });
    }

    @Override
    protected void takeDown() {
        System.out.println(getLocalName() + " : takeDown() -> Terminaison de l'agent observateur.");
    }
}
```

### 4. Main.java (Point d'entrée)

```java
package agents;

import jade.core.Profile;          // Configuration du conteneur JADE
import jade.core.ProfileImpl;      // Implémentation par défaut du profil
import jade.wrapper.AgentController;      // Contrôleur pour gérer un agent
import jade.wrapper.ContainerController;  // Contrôleur pour gérer un conteneur
import jade.core.Runtime;         // Runtime JADE

/**
 * CLASSE MAIN
 * Rôle : Point d'entrée du programme, responsable de :
 * - Initialiser la plateforme JADE
 * - Créer le conteneur principal
 * - Instancier et démarrer les agents
 */
public class Main {
    public static void main(String[] args) {
        try {
            System.out.println("=== INITIALISATION DE LA PLATEFORME JADE ===");
            
            // ÉTAPE 1 : Obtenir l'instance unique du runtime JADE
            // Runtime = environnement d'exécution pour les agents
            Runtime rt = Runtime.instance();
            
            // ÉTAPE 2 : Créer un profil de configuration
            Profile profile = new ProfileImpl();
            
            // Configuration du profil :
            profile.setParameter(Profile.GUI, "true");        // Interface graphique activée
            profile.setParameter(Profile.MAIN_HOST, "localhost"); // Adresse du serveur
            profile.setParameter(Profile.MAIN_PORT, "1099");     // Port de communication
            
            // ÉTAPE 3 : Créer le conteneur principal
            // Le conteneur = environnement où vivent les agents
            ContainerController container = rt.createMainContainer(profile);
            
            System.out.println("=== Conteneur principal créé ===");
            System.out.println("=== Démarrage des agents ===");
            
            // ÉTAPE 4 : Créer les agents
            // Paramètres : createNewAgent(nom_agent, classe_agent, arguments)
            AgentController vendeur = container.createNewAgent("vendeur", 
                "agents.AgentVendeur", null);
            AgentController acheteur = container.createNewAgent("acheteur", 
                "agents.AgentAcheteur", null);
            AgentController observateur = container.createNewAgent("observateur", 
                "agents.AgentObservateur", null);

            // ÉTAPE 5 : Démarrer les agents (appelle automatiquement setup())
            vendeur.start();
            System.out.println("✓ Agent vendeur démarré");
            
            acheteur.start();
            System.out.println("✓ Agent acheteur démarré");
            
            observateur.start();
            System.out.println("✓ Agent observateur démarré");
            
            System.out.println("=== Tous les agents sont opérationnels ===");
            System.out.println("=== Interface graphique JADE disponible ===");
            
        } catch (Exception e) {
            System.err.println("❌ Erreur lors du démarrage du système:");
            e.printStackTrace();
        }
    }
}
```

---

## 📚 Concepts Clés Expliqués

### 1. Cycle de vie d'un agent JADE
```
Création → setup() → Comportements actifs → takeDown() → Destruction
    ↑           ↑              ↑                ↑            ↑
    |           |              |                |            |
Instanciation  Init.     Exécution des     Terminaison   Libération
de l'objet   ressources   behaviours        propre       mémoire
```

### 2. Types de Comportements (Behaviours)

| Type | Description | Utilisation |
|------|-------------|-------------|
| **OneShotBehaviour** | S'exécute une seule fois | Tâches ponctuelles (envoi initial) |
| **CyclicBehaviour** | Boucle infinie | Écoute permanente de messages |
| **TickerBehaviour** | Exécution périodique | Surveillance, monitoring |
| **WakerBehaviour** | Exécution différée | Tâches programmées |

### 3. Communication entre Agents

```java
// Structure d'un message ACL
ACLMessage message = new ACLMessage(performative);
message.addReceiver(destinataire);           // Qui reçoit ?
message.setContent(contenu);                 // Quoi ?
message.setLanguage(langage);                // Dans quel langage ?
message.setOntology(ontologie);              // Selon quelle ontologie ?
send(message);                               // Envoi
```

### 4. Performatives ACL Courantes

| Performative | Signification | Usage |
|--------------|---------------|--------|
| **REQUEST** | Demande d'action | "Peux-tu faire X ?" |
| **INFORM** | Information | "Voici une information" |
| **QUERY_IF** | Question oui/non | "Est-ce que X est vrai ?" |
| **PROPOSE** | Proposition | "Je propose de faire X" |
| **ACCEPT_PROPOSAL** | Acceptation | "J'accepte ta proposition" |

---


## 📊 Résultat Attendu

```
=== INITIALISATION DE LA PLATEFORME JADE ===
=== Conteneur principal créé ===
=== Démarrage des agents ===
✓ Agent vendeur démarré
✓ Agent acheteur démarré
✓ Agent observateur démarré
=== Tous les agents sont opérationnels ===
vendeur : setup() -> Initialisation du vendeur.
acheteur : setup() -> Initialisation de l'acheteur.
observateur : setup() -> Initialisation de l'observateur.
acheteur a envoyé une demande au vendeur.
vendeur a reçu : Bonjour, je veux acheter un produit.
vendeur a envoyé une réponse.
acheteur a reçu la réponse : Réponse du vendeur : Produit disponible !
observateur : Observation du système à Tue Oct 15 11:17:14 UTC 2025
observateur : Observation du système à Tue Oct 15 11:17:17 UTC 2025
...
```

---



**🎓 Fin du TP - Bonne programmation !**
