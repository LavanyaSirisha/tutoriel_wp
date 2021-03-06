Pour terminer cette partie nous allons parler de la composition des appels de
fonctions et commencer à entrer dans les détails de fonctionnement de WP. Nous
allons en profiter pour regarder comment se traduit le découpage de nos 
programmes en fichiers lorsque nous voulons les spécifier et les prouver avec WP.

Notre but sera de prouver la fonction ```max_abs``` qui renvoie les maximums 
entre les valeurs absolues de deux valeurs :

```c
int max_abs(int a, int b){
  int abs_a = abs(a);
  int abs_b = abs(b);

  return max(abs_a, abs_b);
}
```

Commençons par (sur-)découper nos précédentes fonctions en couples 
headers/source pour ```abs``` et ```max```. Cela donne pour ```abs``` :

Fichier abs.h :

```c
#ifndef _ABS
#define _ABS

#include <limits.h>

/*@
  requires val > INT_MIN;
  assigns  \nothing;

  behavior pos:
    assumes 0 <= val;
    ensures \result == val;
  
  behavior neg:
    assumes val < 0;
    ensures \result == -val;
 
  complete behaviors;
  disjoint behaviors;
*/
int abs(int val);

#endif
```

Fichier abs.c

```c
#include "abs.h"

int abs(int val){
  if(val < 0) return -val;
  return val;
}
```

Nous découpons en mettant le contrat de la fonction dans le header. Le but de
ceci est de pouvoir, lorsque nous aurons besoin de la fonction dans un autre 
fichier, importer la spécification en même temps que la déclaration de 
celle-ci. En effet, WP en aura besoin pour montrer que les appels à cette 
fonction sont valides.

Nous pouvons créer un fichier sous le même formatage pour la fonction ```max```.
Dans les deux cas, nous pouvons ré-ouvrir le fichier source (pas besoin de 
spécifier les fichiers headers dans la ligne de commande) avec Frama-C et 
remarquer que la spécification est bien associée à la fonction et que nous
pouvons la prouver.

Maintenant, nous pouvons préparer le terrain pour la fonction ```max_abs```. 
Dans notre header :

```c
#ifndef _MAX_ABS
#define _MAX_ABS

int max_abs(int a, int b);

#endif
```

Et dans le source :

```c
#include "max_abs.h"
#include "max.h"
#include "abs.h"

int max_abs(int a, int b){
  int abs_a = abs(a);
  int abs_b = abs(b);

  return max(abs_a, abs_b);
}
```

Et ouvrir ce dernier fichier dans Frama-C. Si nous regardons le panneau latéral, 
nous pouvons voir que les fichiers header que nous avons inclus dans le fichier 
```abs_max.c``` y apparaissent et que les contrats de fonction sont décorés avec des 
pastilles particulières (vertes et bleues) :

![Le contrat de ```max``` est valide par hypothèse](https://zestedesavoir.com:443/media/galleries/2584/792fb2f6-435f-43ff-adc7-a981ae56f44a.png)

Ces pastilles nous disent qu'en l'absence d'implémentation, les propriétés sont
supposées vraies. Et c'est une des forces de la preuve déductive de programmes 
par rapport à certaines autres méthodes formelles, les fonctions sont vérifiées
en isolation les unes des autres. 

En dehors de la fonction, sa spécification est considérée comme étant 
vérifiée : nous ne cherchons pas à reprouver que la fonction fait bien son travail
à chaque appel, nous nous contenterons de vérifier que les pré-conditions sont 
réunies au moment de l'appel. Cela donne donc des preuves très modulaires et donc 
des spécifications plus facilement réutilisables. Évidemment, si notre preuve 
repose sur la spécification d'une autre fonction, cette fonction doit-elle même 
être vérifiable pour que la preuve soit formellement complète. Mais nous pouvons
également vouloir simplement faire confiance à une bibliothèque externe sans la
prouver.

Finalement, le lecteur pourra essayer de spécifier la fonction ```max_abs```.

La spécification peut ressembler à ceci (j'ai mis l'implémentation avec pour
rappel) :

```c
/*@
  requires a > INT_MIN;
  requires b > INT_MIN;

  assigns \nothing;

  ensures \result >= 0;
  ensures \result >= a && \result >= -a && \result >= b && \result >= -b;
  ensures \result == a || \result == -a || \result == b || \result == -b;
*/
int abs_max(int a, int b){
  int abs_a = abs(a);
  int abs_b = abs(b);

  return max(abs_a, abs_b);
}
```
