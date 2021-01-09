# Render Pass

Un render pass \(passe de rendu en francais\) est une description générale des étapes de nos commandes de dessin, que nous verrons par la suite, ces étapes sont divisées en ressources utilisées pendant le rendu. Nous ne pouvons rien rendre dans Vulkan sans une passe de rendu. Et chaque passe de rendu doit comporter une ou plusieurs étapes. Ces étapes sont appelées,

## Description de l'attachment

## Référence de attachment

## Subpasses

Les sous-passes et chaque sous-passe utilisent une \(sous\) collection de ressources définies pour la passe de rendu. Les ressources de la passe de rendu peuvent inclure des cibles de rendu \(couleur, profondeur / gabarit, résolution\) et des données d'entrée \(ressources qui, potentiellement, étaient des cibles de rendu dans les sous-passes précédentes de la même passe de rendu\). Et ces ressources s'appellent, attachment \(elles n'incluent pas les descripteurs / textures / samplers et buffers\).

Pourquoi ne les appelons-nous pas simplement des cibles de rendu ou des images ? Parce que nous ne les rendons pas seulement \(input attachment\) et parce que ce ne sont que des descriptions \(méta-données\). Les images qui doivent être utilisées comme attachments à l'intérieur des passes de rendu sont fournies via des buffers d'image.

## Dépendance

## Création du render pass

