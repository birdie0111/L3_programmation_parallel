# Rapport du projet de Programmation parallèle



**Antoine Dumoulin** & **Qinyue Liu**, L3 informatique




# Environnement de machine

Le système utilisé pour réaliser les tests et les mesures de performances est :

```
OS : Linux tendoLaptop 5.4.0-53-generic #59~18.04.1-Ubuntu SMP Wed Oct 21 12:14:56 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
OS dans un dualboot
#processeurs: 4   
intel hyper-threading : Thread(s) per core:  2
compilateur: 
- gcc (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0
variable d'environnement:
OMP_NUM_THREADS=4
```


# Explications

Cette commande regroupe toutes les lignes qui ont été ajoutées :
```grep -rn --exclude=rapport.md pragma```

Dans le fichier **solution_check.c**:

- dans la fonction `solution_check`:
On calcule le nombre d'erreurs. La variable `errors` est la seule qui y est modifiée. On effectue alors une reduction dessus lors de la 
parallelisation de la boucle for.
La fonction standard `qsort` n'est pas parallélisable trivialement. Il fallait appeler
la fonction `street_table_sort_ids` avant la zone parallèle.
La fonction `solution_check` n'étant pas la plus importante, cette parallèlisation 
apporte un speedup moindre.

- dans la fonction `simulation_dequeue`:
On cherche entre toutes les rues de p pour vérifier s'il reste des rues à enlever. On utilise ```pragma omp for``` pour optimiser notre cherche dans la boucle for.
Une voiture ne peut être que dans une rue à la fois, donc il n'y a pas de concurrence.

- dans la fonction `simulation_run`:
On démarre la région parallèle dans la boucle qui simule le temps. On y calcule les notes. La variable `score` sera incrémentée, donc on effectue une reduction.
Dans cette boucle, on partage l'exécution de la fonction `simulation_update_intersection_lights` dans une boucle avec simplement ```pragma omp for``` car les feux ne sont pas dépendants.
Par contre les voitures sont dépendantes dans la fonction `simulation_update_car`, alors en plus d'utiliser ```pragma omp for```, il y a deux sections critiques dans cette fonction.


# Mesures de performances

./checker input/a.txt out/a_best.out
Score 2002 - OK
Reference time : 0,002s
Kernel time -- : 00.01s

./checker input/b.txt out/b_best.out
Score 4565760 - OK
Reference time : 1,294s
Kernel time -- : 00.37s

./checker input/c.txt out/c_best.out
Score 1297997 - OK
Reference time : 0,958s
Kernel time -- : 00.39s

./checker input/d.txt out/d_best.out
Score 1556815 - OK
Reference time : 10,707s
Kernel time -- : 05.23s

./checker input/e.txt out/e_best.out
Score 690171 - OK
Reference time : 0,024s
Kernel time -- : 00.01s

./checker input/f.txt out/f_best.out
Score 1356372 - OK
Reference time : 0,363s
Kernel time -- : 00.24s

./checker input/blong.txt out/blong_best.out
Score 10606842 - OK
Reference time : 2,816s
Kernel time -- : 00.69s

./checker input/clong.txt out/clong_best.out
Score 19657184 - OK
Reference time : 9,071s
Kernel time -- : 02.75s

./checker input/dlong.txt out/dlong_best.out
Score 11580716 - OK
Reference time : 18,044s
Kernel time -- : 8.56s

**Somme des fichiers**

Reference time : 41,2s
Kernel time -- : 18.3s
Speedup ------ : 2.27
Efficiency --- : 0,56

*Commentaires*

Le speedup varie en fonction du fichier d'input. La taille et la forme des données
peuvent en effet faire varier le pourcentage de code parallèlisé exécuté.
La parallélisation est moyennement efficace.


# Suggestions pour gagner en efficacité

Même si nous sommes limités par le code nécessairement séquentiel, certaines parties auraient pu être améliorées :

- La fonction `qsort`:
On aurait pu recoder la fonction qsort pour pouvoir profiter du parallèlisme. Pour ce faire on peut diviser le tri de la table pour effectuer un tri par fusion des tableaux triés parallèlement.

- Dans la fonction `simulation_update_car`:
Les zones critiques pourraient être évités lorsque les `car_state[c].street` sont différents. Il aurait alors été judiciable d'ajouter un mutex dans la structure de donnée `street_table`.




