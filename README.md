# TP JADE : Multi-Agents avec cycle de vie

**Exemple avec trois agents et méthode main**

---

**Préparé par :** EL AOUMARI Abdelmoughith & LEMKHARBECH Yahya

**Université :** Cadi Ayyad - FS–Semlalia–Marrakech  
**Module :** IAD & I. Connaissances  
**Professeur :** M. Qaffou Issam  
**Année universitaire :** 2025–2026

---

## Introduction

Ce TP montre comment créer plusieurs agents JADE, comment ils communiquent via ACL, et illustre le cycle de vie d'un agent JADE :

- **setup()** : initialisation
- **action()** : exécution répétitive ou unique selon le comportement
- **takeDown()** : terminaison

Une classe `Main` permet de lancer les agents depuis Java directement.

## I. Création des agents

### I.1 AgentVendeur.java

```java
package agents;

import jade.core.Agent;
import jade.core.behaviours.CyclicBehaviour;
import jade.lang.acl.ACLMessage;

public class AgentVendeur extends Agent {

    @Override
    protected void setup() {
        System.out.println(getLocalName() + " : setup() -> Initialisation.");
        addBehaviour(new CyclicBehaviour() {
            @Override
            public void action() {
                ACLMessage msg = receive();
                if (msg != null) {
                    System.out.println(getLocalName() + " a reçu : " + msg.getContent());
                    ACLMessage reply = msg.createReply();
                    reply.setPerformative(ACLMessage.INFORM);
                    reply.setContent("Réponse du vendeur : Produit disponible !");
                    send(reply);
                } else {
                    block();
                }
            }
        });
    }

    @Override
    protected void takeDown() {
        System.out.println(getLocalName() + " : takeDown() -> Terminaison de l'agent.");
    }
}
```

### I.2 AgentAcheteur.java

```java
package agents;

import jade.core.Agent;
import jade.core.behaviours.OneShotBehaviour;
import jade.lang.acl.ACLMessage;

public class AgentAcheteur extends Agent {

    @Override
    protected void setup() {
        System.out.println(getLocalName() + " : setup() -> Initialisation.");
        addBehaviour(new OneShotBehaviour() {
            @Override
            public void action() {
                ACLMessage msg = new ACLMessage(ACLMessage.REQUEST);
                msg.addReceiver(getContainerController().getAgent("vendeur").getAID());
                msg.setContent("Bonjour, je veux acheter un produit.");
                send(msg);
            }
        });
    }

    @Override
    protected void takeDown() {
        System.out.println(getLocalName() + " : takeDown() -> Terminaison de l'agent.");
    }
}
```

### I.3 AgentObservateur.java

```java
package agents;

import jade.core.Agent;
import jade.core.behaviours.CyclicBehaviour;
import jade.lang.acl.ACLMessage;

public class AgentObservateur extends Agent {

    @Override
    protected void setup() {
        System.out.println(getLocalName() + " : setup() -> Initialisation.");
        addBehaviour(new CyclicBehaviour() {
            @Override
            public void action() {
                ACLMessage msg = receive();
                if (msg != null) {
                    System.out.println(getLocalName() + " observe : " 
                        + msg.getSender().getLocalName() + " -> " + msg.getContent());
                } else {
                    block();
                }
            }
        });
    }

    @Override
    protected void takeDown() {
        System.out.println(getLocalName() + " : takeDown() -> Terminaison de l'agent.");
    }
}
```

## II. Classe Main pour lancer les agents depuis Java

```java
package agents;

import jade.core.Profile;
import jade.core.ProfileImpl;
import jade.wrapper.AgentController;
import jade.wrapper.ContainerController;
import jade.core.Runtime;

public class Main {
    public static void main(String[] args) {
        Runtime rt = Runtime.instance();
        Profile profile = new ProfileImpl();
        profile.setParameter(Profile.GUI, "true"); // pour ouvrir le GUI JADE
        ContainerController container = rt.createMainContainer(profile);

        try {
            AgentController vendeur = container.createNewAgent("vendeur", "agents.AgentVendeur", null);
            AgentController acheteur = container.createNewAgent("acheteur", "agents.AgentAcheteur", null);
            AgentController observateur = container.createNewAgent("observateur", "agents.AgentObservateur", null);

            vendeur.start();
            acheteur.start();
            observateur.start();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## III. Cycle de vie des agents JADE

1. **Construction** : création de l'instance de l'agent
2. **setup()** : initialisation des ressources et comportements
3. **action()** : exécution des comportements (OneShotBehaviour, CyclicBehaviour, etc.)
4. **suspension** : l'agent peut être bloqué si aucun message ou événement n'est reçu
5. **takeDown()** : terminaison et nettoyage des ressources

## Conclusion

Ce TP montre :

- Comment créer plusieurs agents avec JADE
- Comment gérer le cycle de vie des agents
- Comment lancer les agents depuis une méthode `main` en Java
- Comment réaliser des communications ACL et observer les échanges avec un agent observateur

---

**Intelligence Artificielle Distribuée - Ingénierie des Connaissances**
