#VMelnikDoctrineEncryptBundle Example Of Usage

Lets imagine that you storing some private data in your database and don't want to somebody can see it 
event if he has raw database on the hands in some dirty way. With this bundle this task can be 
easily made and you even don't see these processes because bundle uses some doctrine life cycle 
events and in database information will be encoded and your entities in program will be clear as 
always and all these things will be happen automatically.

## Simple example

For example, we have some user entity with two fields which we want to encode in database. This entity 
must implement `VMelnik\DoctrineEncryptBundle\Encryptors\EncryptableInterface` interface and has method 
`getEncryptedFields` wich will return array with fields names needed in encode\decode operations.

###Doctrine Entity

```php
namespace Acme\DemoBundle\Entity;

use Doctrine\ORM\Mapping as ORM;
use VMelnik\DoctrineEncryptBundle\Encryptors\EncryptableInterface;

/**
 * @ORM\Entity
 * @ORM\Table(name="user_v")
 */
class UserV implements EncryptableInterface {
    
    /**
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="AUTO")
     * @ORM\Column(type="integer")
     * @var int
     */
    private $id;
    
    /**
     * @ORM\Column(type="text", name="total_money")
     * @var int
     */
    private $totalMoney;
    
    /**
     * @ORM\Column(type="string", length=100, name="first_name")
     * @var string
     */
    private $firstName;
    
    /**
     * @ORM\Column(type="string", length=100, name="last_name")
     * @var string
     */
    private $lastName;
    
    /**
     * @ORM\Column(type="text", name="credit_card_number")
     * @var string
     */
    private $creditCardNumber;
    
    //common getters/setters here...

    // we implement method of EncryptableInterface and return two encoded fields names
    public function getEncryptedFields() {
        return array('totalMoney', 'creditCardNumber');
    }
    
}
```

###Fixtures

```php

namespace Acme\DemoBundle\DataFixtures\ORM;

use Doctrine\Common\Persistence\ObjectManager;
use Doctrine\Common\DataFixtures\FixtureInterface;
use Acme\DemoBundle\Entity\UserV;

class LoadUserData implements FixtureInterface
{
    public function load(ObjectManager $manager)
    {
        $user = new UserV();
        $user->setFirstName('Victor');
        $user->setLastName('Melnik');
        $user->setTotalMoney('1000000000000');
        $user->setCreditCardNumber('1234567890');

        $manager->persist($user);
        $manager->flush();
    }
}
```

###Controller

```php

namespace Acme\DemoBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;

// these import the "@Route" and "@Template" annotations
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

// our entity
use Acme\DemoBundle\Entity\UserV;

class DemoController extends Controller
{
    /**
     * @Route("/show-user/{id}", name="_vmelnik_decrypt_test", requirements={"id" = "\d+"})
     * @Template
     */
    public function getUserAction(UserV $user) {}
}
```

###Template

```twig
<div>Common info: {{ user.lastName ~  ' ' ~ user.firstName }}</div>
<div>
    Decoded info:
    <dl>
        <dt>Total money<dt>
        <dd>{{ user.totalMoney }}</dd>
        <dt>Credit card<dt>
        <dd>{{ user.creditCardNumber }}</dd>
    </dl>
</div> 
```

When we follow link <site>/show-user/1 we will see that all user's information is decoded 
like in fixture, but in the same time information in database will something like this:

+----+----------------------------------------------+------------+-----------+----------------------------------------------+
| id | total_money                                  | first_name | last_name | credit_card_number                           |
+----+----------------------------------------------+------------+-----------+----------------------------------------------+
|  1 | dx+taMIxyUdI3OTlqkjDBKRWP9Qr28PCaCCYxwbjEQU= | Victor     | Melnik    | 1Y+Yzq6/dDXvtnYHhTyadWfIm6xhGLxuKL2oSuxuzL4= |
+----+----------------------------------------------+------------+-----------+----------------------------------------------+

So your information is encoded and all okay.

###Requirements

You need `DoctrineFixturesBundle`, `php-mcrypt` extension