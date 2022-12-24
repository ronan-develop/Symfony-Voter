# Symfony Voter
⚠️ Cette classe ne fait que renvoyer `true` or `false` pour déterminer si accès ou pas. ⚠️

C'est une centralisation de la logique d'autorisation qui peut être
réutilisée à de nombreux endroits dans l'application.

Un `Voter` personnalisé doit implémenter la VoterInterface ou `extends Voter`

Détail du contrat de l'interface pour les curieux :
```php
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Authorization\Voter\VoterInterface;

abstract class Voter implements VoterInterface
{
    abstract protected function supports(string $attribute, mixed $subject): bool;
    abstract protected function voteOnAttribute(string $attribute, mixed $subject, TokenInterface $token): bool;
}
```
Ce contrat implique l'utilisation des méthodes [supports](#supports) et [voteOnAttribute](#voteonattribute).
Elles seront détaillées ci-après.

## Controller
Dans la partie controller, pour le traitement de la requête, oin doit vérifier l'accès avec la méthode 
`$this->denyAccessUnlessGranted('une-règle', 'instance-a-controller');`

Imaginons un blog pour lequel on veut controller l'accès aux post :
```php
class PostController extends AbstractController
{
    #[Route('/posts/{id}', name: 'post_show')]
    public function show($id): Response
    {
        // get a Post object - e.g. query for it
        $post = 'Requête_DataBase';

        // check for "view" access: calls all voters
        $this->denyAccessUnlessGranted('view', $post);

        // ...
    }
    ...
}
```
`denyAccessUnlessGranted` provient de l'AbstractController.  
<u>Le premier</u> paramètre de cette méthode et le nom de la règle à suivre, que l'on retrouvera sous
la forme `$attibute` dans le voter.

<u>Le deuxième</u> paramètre est l'instance de l'entité pour laquelle on veut controller l'accès.

## Voter
Le plus simple et de créer cette classe avec le maker de [Symfony](https://symfony.com/bundles/SymfonyMakerBundle/current/index.html).
```bash
php bin/console list make

make:command            Creates a new console command class
make:controller         Creates a new controller class
make:entity             Creates a new Doctrine entity class

[...]

make:validator          Creates a new validator and constraint class
make:voter              Creates a new security voter class
```
Un Voter générique qui respectera le contrat de l'interface sera automatiquement créé.
```php
<?php

namespace App\Security\Voter;

use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Authorization\Voter\Voter;
use Symfony\Component\Security\Core\User\UserInterface;

class TaskVoter extends Voter
{
    public const EDIT = 'edit';
    public const VIEW = 'view';

    protected function supports(string $attribute, mixed $subject): bool
    {
        // replace with your own logic
        // https://symfony.com/doc/current/security/voters.html
        return in_array($attribute, [self::EDIT, self::VIEW])
            && $subject instanceof \App\Entity\Task;
    }

    protected function voteOnAttribute(string $attribute, mixed $subject, TokenInterface $token): bool
    {
        $user = $token->getUser();
        // if the user is anonymous, do not grant access
        if (!$user instanceof UserInterface) {
            return false;
        }

        // ... (check conditions and return true to grant permission) ...
        switch ($attribute) {
            case self::EDIT:
                // logic to determine if the user can EDIT
                // return true or false
                break;
            case self::VIEW:
                // logic to determine if the user can VIEW
                // return true or false
                break;
        }

        return false;
    }
}
```
Les constantes sont "rangées" dans un tableau.
### Supports
La méthode `supports()` va commencer par vérifier, si le paramètre $attribute
passé avec le controller est bien présent dans le voter <u>ET</u> que $subject est bien une instance
de ce que l'on souhaite vérifier.
```php
protected function supports(string $attribute, mixed $subject): bool
{
    // replace with your own logic
    // https://symfony.com/doc/current/security/voters.html
    return in_array($attribute, [self::EDIT, self::VIEW])
        && $subject instanceof \App\Entity\MonEntité;
}
```
Si non, return false.
### VoteOnAttribute
Comme dans la classe abstraite `Voter`, la méthode `vote(TokenInterface $token, mixed $subject, array $attributes)`
est appelée, cette méthode exécute également notre `voteOnAttribute(string $attribute, mixed $subject, TokenInterface $token)`. Le travail de cette méthode est de vérifier que l'utilisateur en session, récupéré
grâce à la `TokenInterface`:
```php
$user = $token->getUser();
```
soit bien une instance `UserInterface`.
Le switch ou match qui suit va déterminer les cas possibles pour le $attribute passé.
```php
switch ($attribute) {
    case self::EDIT:
        // logic to determine if the user can EDIT
        // return true or false
        $this->uneMethode()
    case self::VIEW:
        // logic to determine if the user can VIEW
        // return true or false
        $this->uneAutreMethode()
}
return false;

private function uneMethode(){
// placer ici de la logique return true or false
}

private function private function uneMethode(){
// placer ici de la logique return true or false
}
```

