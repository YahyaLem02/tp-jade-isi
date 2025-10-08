# TP JADE : Multi-Agents avec cycle de vie - Version Corrigée

**Exemple avec trois agents et méthode main**

---

**Préparé par :** EL AOUMARI Abdelmoughith & LEMKHARBECH Yahya

## Structure du projet

```
projet-jade/
├── src/
│   └── agents/
│       ├── AgentVendeur.java
│       ├── AgentAcheteur.java
│       ├── AgentObservateur.java
│       └── Main.java
└── lib/
    └── jade.jar
```

## Fichiers corrigés

### 1. AgentVendeur.java

```java
package agents;

import jade.core.Agent;
import jade.core.AID;
import jade.core.behaviours.CyclicBehaviour;
import jade.lang.acl.ACLMessage;

public class AgentVendeur extends Agent {

    @Override
    protected void setup() {
        System.out.println(getLocalName() + " : setup() -> Initialisation du vendeur.");
        
        addBehaviour(new CyclicBehaviour() {
            @Override
            public void action() {
                ACLMessage msg = receive();
                if (msg != null) {
                    System.out.println(getLocalName() + " a reçu : " + msg.getContent());
                    
                    // Créer une réponse
                    ACLMessage reply = msg.createReply();
                    reply.setPerformative(ACLMessage.INFORM);
                    reply.setContent("Réponse du vendeur : Produit disponible !");
                    send(reply);
                    
                    System.out.println(getLocalName() + " a envoyé une réponse.");
                } else {
                    block();
                }
            }
        });
    }

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
import jade.core.behaviours.OneShotBehaviour;
import jade.core.behaviours.CyclicBehaviour;
import jade.lang.acl.ACLMessage;

public class AgentAcheteur extends Agent {

    @Override
    protected void setup() {
        System.out.println(getLocalName() + " : setup() -> Initialisation de l'acheteur.");
        
        // Comportement pour envoyer un message
        addBehaviour(new OneShotBehaviour() {
            @Override
            public void action() {
                // Attendre un peu que le vendeur soit initialisé
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                
                ACLMessage msg = new ACLMessage(ACLMessage.REQUEST);
                msg.addReceiver(new AID("vendeur", AID.ISLOCALNAME));
                msg.setContent("Bonjour, je veux acheter un produit.");
                send(msg);
                
                System.out.println(getLocalName() + " a envoyé une demande au vendeur.");
            }
        });
        
        // Comportement pour recevoir les réponses
        addBehaviour(new CyclicBehaviour() {
            @Override
            public void action() {
                ACLMessage msg = receive();
                if (msg != null) {
                    System.out.println(getLocalName() + " a reçu la réponse : " + msg.getContent());
                } else {
                    block();
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
import jade.core.behaviours.TickerBehaviour;

public class AgentObservateur extends Agent {

    @Override
    protected void setup() {
        System.out.println(getLocalName() + " : setup() -> Initialisation de l'observateur.");
        
        // Comportement d'observation périodique
        addBehaviour(new TickerBehaviour(this, 3000) { // Toutes les 3 secondes
            @Override
            protected void onTick() {
                System.out.println(getLocalName() + " : Observation du système à " + 
                    new java.util.Date());
            }
        });
    }

    @Override
    protected void takeDown() {
        System.out.println(getLocalName() + " : takeDown() -> Terminaison de l'agent observateur.");
    }
}
```

### 4. Main.java (Corrigé)

```java
package agents;

import jade.core.Profile;
import jade.core.ProfileImpl;
import jade.wrapper.AgentController;
import jade.wrapper.ContainerController;
import jade.core.Runtime;

public class Main {
    public static void main(String[] args) {
        try {
            // Obtenir l'instance du runtime JADE
            Runtime rt = Runtime.instance();
            
            // Créer un profil pour le conteneur principal
            Profile profile = new ProfileImpl();
            profile.setParameter(Profile.GUI, "true"); // Activer l'interface graphique
            profile.setParameter(Profile.MAIN_HOST, "localhost");
            profile.setParameter(Profile.MAIN_PORT, "1099");
            
            // Créer le conteneur principal
            ContainerController container = rt.createMainContainer(profile);
            
            System.out.println("=== Démarrage des agents ===");
            
            // Créer et démarrer les agents
            AgentController vendeur = container.createNewAgent("vendeur", 
                "agents.AgentVendeur", null);
            AgentController acheteur = container.createNewAgent("acheteur", 
                "agents.AgentAcheteur", null);
            AgentController observateur = container.createNewAgent("observateur", 
                "agents.AgentObservateur", null);

            // Démarrer les agents
            vendeur.start();
            System.out.println("Agent vendeur démarré");
            
            acheteur.start();
            System.out.println("Agent acheteur démarré");
            
            observateur.start();
            System.out.println("Agent observateur démarré");
            
            System.out.println("=== Tous les agents sont démarrés ===");
            
        } catch (Exception e) {
            System.err.println("Erreur lors du démarrage des agents:");
            e.printStackTrace();
        }
    }
}
```

## Instructions de compilation et exécution

### 1. Prérequis
- Java JDK 8 ou supérieur
- Bibliothèque JADE (jade.jar)

### 2. Compilation
```bash
# Naviguer vers le répertoire du projet
cd projet-jade

# Compiler les classes Java
javac -cp "lib/jade.jar" src/agents/*.java

# Ou sur Windows
javac -cp "lib\jade.jar" src\agents\*.java
```

### 3. Exécution
```bash
# Exécuter depuis le répertoire src
cd src
java -cp ".;../lib/jade.jar" agents.Main

# Ou sur Linux/Mac
java -cp ".:../lib/jade.jar" agents.Main
```

### 4. Alternative avec IDE
Si vous utilisez un IDE (Eclipse, IntelliJ, etc.) :
1. Créez un nouveau projet Java
2. Ajoutez jade.jar au classpath du projet
3. Créez le package `agents`
4. Ajoutez les fichiers Java
5. Exécutez la classe Main

## Cycle de vie des agents JADE

1. **Construction** : création de l'instance de l'agent
2. **setup()** : initialisation des ressources et comportements
3. **action()** : exécution des comportements (OneShotBehaviour, CyclicBehaviour, etc.)
4. **suspension** : l'agent peut être bloqué si aucun message ou événement n'est reçu
5. **takeDown()** : terminaison et nettoyage des ressources

## Résultat attendu

Après exécution, vous devriez voir :
```
=== Démarrage des agents ===
Agent vendeur démarré
Agent acheteur démarré
Agent observateur démarré
=== Tous les agents sont démarrés ===
vendeur : setup() -> Initialisation du vendeur.
acheteur : setup() -> Initialisation de l'acheteur.
observateur : setup() -> Initialisation de l'observateur.
acheteur a envoyé une demande au vendeur.
vendeur a reçu : Bonjour, je veux acheter un produit.
vendeur a envoyé une réponse.
acheteur a reçu la réponse : Réponse du vendeur : Produit disponible !
observateur : Observation du système à [date/heure]
```

**Intelligence Artificielle Distribuée - Ingénierie des Connaissances**
