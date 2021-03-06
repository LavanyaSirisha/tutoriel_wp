Les lemmes sont des propriétés générales à propos des prédicats ou encore des 
fonctions. Une fois ces propriétés exprimées, la preuve peut être réalisée une 
fois et la propriété en question pourra être utilisée par les prouveurs, leur 
permettant ainsi de ne pas reproduire les étapes de preuve nécessaires à chaque
fois qu'une propriété équivalente intervient dans une preuve plus longue sur 
une propriété plus précise.

Les lemmes peuvent par exemple nous permettre d'exprimer des propriétés à 
propos des fonctions récursives pour que les preuves les faisant intervenir 
nécessitent moins de travail pour les prouveurs.

# Syntaxe

Une nouvelle fois, nous les introduisons à l'aide d'annotations ACSL. La syntaxe
utilisée est la suivante :

```c
/*@
  lemma name_of_the_lemma { Label0, ..., LabelN }:
    property ;
*/
```

Cette fois les propriétés que nous voulons exprimer ne dépendent pas de 
paramètres reçus (hors de nos *labels* bien sûr). Ces propriétés seront donc 
exprimées sur des variables quantifiées. Par exemple, nous pouvons poser ce 
lemme qui est vrai, même s'il est trivial :

```c
/*@
  lemma lt_plus_lt:
    \forall integer i, j ; i < j ==> i+1 < j+1;
*/
```

Cette preuve peut être effectuée en utilisant WP. La propriété est bien sûr 
trivialement prouvée par Qed.

# Exemple : propriété fonction affine

Nous pouvons par exemple reprendre nos fonctions affines et exprimer quelques 
propriétés intéressantes à leur sujet :

```c
/* @
  lemma ax_b_monotonic_neg:
    \forall integer a, b, i, j ;
      a <  0 ==> i <= j ==> ax_b(a, i, b) >= ax_b(a, j, b);
  lemma ax_b_monotonic_pos:
    \forall integer a, b, i, j ;
      a >  0 ==> i <= j ==> ax_b(a, i, b) <= ax_b(a, j, b);
  lemma ax_b_monotonic_nul:
    \forall integer a, b, i, j ;
      a == 0 ==> ax_b(a, i, b) == ax_b(a, j, b);
*/
```

Pour ces preuves, il est fort possible qu'Alt-ergo ne parvienne pas à les 
décharger. Dans ce cas, le prouveur Z3 devrait, lui, y arriver. Nous pouvons 
ensuite construire cet exemple de code :

```c
/*@
  requires a > 0;
  requires limit_int_min_ax_b(a,4) < x < limit_int_max_ax_b(a,4);
  assigns \nothing ;
  ensures \result == ax_b(a,x,4);
*/
int function(int a, int x){
  return a*x + 4;
}

/*@ 
  requires a > 0;
  requires limit_int_min_ax_b(a,4) < x < limit_int_max_ax_b(a,4) ;
  requires limit_int_min_ax_b(a,4) < y < limit_int_max_ax_b(a,4) ;
  assigns \nothing ;
*/
void foo(int a, int x, int y){
  int fmin, fmax;
  if(x < y){
    fmin = function(a,x);
    fmax = function(a,y);
  } else {
    fmin = function(a,y);
    fmax = function(a,x);
  }
  //@assert fmin <= fmax;
}
```

Si nous ne renseignons pas les lemmes mentionnés plus tôt, il y a peu de chances 
qu'Alt-ergo réussisse à produire la preuve que ```fmin``` est inférieur à ```fmax```.
Avec ces lemmes présents en revanche, il y parvient sans problème car cette 
propriété est une simple instance du lemme ```ax_b_monotonic_pos```, la preuve 
étant ainsi triviale car notre lemme nous énonce cette propriété comme étant vraie.
