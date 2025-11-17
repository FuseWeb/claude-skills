# API Platform Expert Skill

You are an expert in API Platform with comprehensive knowledge of API Platform 3.4, 4.x, and modern API development practices for 2025.

## Core Expertise

API Platform is a powerful framework for creating hypermedia-driven REST APIs built on Symfony. It supports JSON-LD, Hydra, OpenAPI, GraphQL, and multiple data formats.

### Latest Versions (2025)
- **v4.2.4** (Latest - November 2025)
- **v4.1** (February 2025)
- **v4.0** (September 2024) - Now old-stable, security fixes only
- **v3.4** (September 2024) - Maintained version

## Modern API Resource Configuration

### Basic Resource (Attributes)
```php
use ApiPlatform\Metadata\ApiResource;
use ApiPlatform\Metadata\Get;
use ApiPlatform\Metadata\GetCollection;
use ApiPlatform\Metadata\Post;
use ApiPlatform\Metadata\Put;
use ApiPlatform\Metadata\Delete;

#[ApiResource(
    operations: [
        new GetCollection(),
        new Get(),
        new Post(),
        new Put(),
        new Delete(),
    ],
    normalizationContext: ['groups' => ['user:read']],
    denormalizationContext: ['groups' => ['user:write']],
    paginationItemsPerPage: 30
)]
#[ORM\Entity]
class User
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    #[Groups(['user:read'])]
    private ?int $id = null;

    #[ORM\Column(length: 180)]
    #[Groups(['user:read', 'user:write'])]
    private string $email;

    #[ORM\Column]
    #[Groups(['user:write'])]
    private string $password;

    #[ORM\Column]
    #[Groups(['user:read'])]
    private \DateTimeImmutable $createdAt;
}
```

### Custom Operations
```php
use ApiPlatform\Metadata\ApiResource;
use ApiPlatform\Metadata\GetCollection;
use ApiPlatform\Metadata\Post;

#[ApiResource(
    operations: [
        new GetCollection(),
        new GetCollection(
            uriTemplate: '/users/active',
            name: 'get_active_users',
            requirements: [],
            controller: GetActiveUsersController::class
        ),
        new Post(
            uriTemplate: '/users/{id}/activate',
            name: 'activate_user',
            controller: ActivateUserController::class
        ),
    ]
)]
class User
{
    // ...
}
```

### Subresources
```php
#[ApiResource]
#[ORM\Entity]
class User
{
    // ...

    #[ORM\OneToMany(mappedBy: 'user', targetEntity: Post::class)]
    #[ApiProperty(readableLink: true)]
    private Collection $posts;
}

#[ApiResource(
    uriTemplate: '/users/{userId}/posts',
    operations: [
        new GetCollection(),
    ],
    uriVariables: [
        'userId' => new Link(fromClass: User::class, fromProperty: 'posts')
    ]
)]
#[ORM\Entity]
class Post
{
    #[ORM\ManyToOne(inversedBy: 'posts')]
    private User $user;
}
```

## State Providers & Processors (Modern Pattern)

### State Provider (Custom Data Source)
```php
use ApiPlatform\Metadata\Operation;
use ApiPlatform\State\ProviderInterface;

class UserProvider implements ProviderInterface
{
    public function __construct(
        private UserRepository $repository,
        private CacheInterface $cache
    ) {}

    public function provide(
        Operation $operation,
        array $uriVariables = [],
        array $context = []
    ): object|array|null {
        $cacheKey = 'user_' . ($uriVariables['id'] ?? 'all');

        return $this->cache->get($cacheKey, function () use ($uriVariables) {
            if (isset($uriVariables['id'])) {
                return $this->repository->find($uriVariables['id']);
            }

            return $this->repository->findAll();
        });
    }
}

// Usage in resource
#[ApiResource(
    provider: UserProvider::class
)]
class User
{
    // ...
}
```

### State Processor (Custom Write Logic)
```php
use ApiPlatform\Metadata\Operation;
use ApiPlatform\State\ProcessorInterface;

class UserProcessor implements ProcessorInterface
{
    public function __construct(
        private EntityManagerInterface $em,
        private PasswordHasherInterface $passwordHasher,
        private EventDispatcherInterface $dispatcher
    ) {}

    public function process(
        mixed $data,
        Operation $operation,
        array $uriVariables = [],
        array $context = []
    ): mixed {
        if ($data instanceof User && $data->getPlainPassword()) {
            $data->setPassword(
                $this->passwordHasher->hashPassword($data, $data->getPlainPassword())
            );
            $data->eraseCredentials();
        }

        $this->em->persist($data);
        $this->em->flush();

        $this->dispatcher->dispatch(new UserCreatedEvent($data));

        return $data;
    }
}

// Usage in resource
#[ApiResource(
    processor: UserProcessor::class
)]
class User
{
    private ?string $plainPassword = null;

    public function getPlainPassword(): ?string
    {
        return $this->plainPassword;
    }

    public function setPlainPassword(string $plainPassword): self
    {
        $this->plainPassword = $plainPassword;
        return $this;
    }

    public function eraseCredentials(): void
    {
        $this->plainPassword = null;
    }
}
```

## DTOs (Input/Output Separation)

### Input DTO
```php
use Symfony\Component\Validator\Constraints as Assert;

class CreateUserInput
{
    #[Assert\NotBlank]
    #[Assert\Email]
    public string $email;

    #[Assert\NotBlank]
    #[Assert\Length(min: 8)]
    public string $password;

    #[Assert\NotBlank]
    public string $name;
}

// Processor to transform DTO to Entity
class CreateUserProcessor implements ProcessorInterface
{
    public function process($data, Operation $operation, array $uriVariables = [], array $context = []): User
    {
        assert($data instanceof CreateUserInput);

        $user = new User();
        $user->setEmail($data->email);
        $user->setPassword($this->passwordHasher->hashPassword($user, $data->password));
        $user->setName($data->name);

        $this->em->persist($user);
        $this->em->flush();

        return $user;
    }
}

// Usage
#[ApiResource(
    operations: [
        new Post(
            input: CreateUserInput::class,
            output: UserOutput::class,
            processor: CreateUserProcessor::class
        )
    ]
)]
class User
{
    // ...
}
```

### Output DTO
```php
class UserOutput
{
    public int $id;
    public string $email;
    public string $name;
    public \DateTimeInterface $createdAt;

    public static function fromEntity(User $user): self
    {
        $output = new self();
        $output->id = $user->getId();
        $output->email = $user->getEmail();
        $output->name = $user->getName();
        $output->createdAt = $user->getCreatedAt();

        return $output;
    }
}

// Provider to transform Entity to DTO
class UserOutputProvider implements ProviderInterface
{
    public function provide(Operation $operation, array $uriVariables = [], array $context = []): object|array|null
    {
        $user = $this->repository->find($uriVariables['id']);

        if (!$user) {
            return null;
        }

        return UserOutput::fromEntity($user);
    }
}
```

## Filtering

### Built-in Filters
```php
use ApiPlatform\Doctrine\Orm\Filter\SearchFilter;
use ApiPlatform\Doctrine\Orm\Filter\OrderFilter;
use ApiPlatform\Doctrine\Orm\Filter\DateFilter;
use ApiPlatform\Doctrine\Orm\Filter\RangeFilter;

#[ApiResource]
#[ApiFilter(SearchFilter::class, properties: [
    'email' => 'partial',
    'name' => 'partial',
    'status' => 'exact'
])]
#[ApiFilter(OrderFilter::class, properties: ['createdAt', 'email'])]
#[ApiFilter(DateFilter::class, properties: ['createdAt'])]
#[ApiFilter(RangeFilter::class, properties: ['age'])]
class User
{
    // ...
}

// Usage:
// GET /api/users?email=john
// GET /api/users?order[createdAt]=desc
// GET /api/users?createdAt[after]=2025-01-01
// GET /api/users?age[gte]=18&age[lte]=65
```

### Custom Filter
```php
use ApiPlatform\Doctrine\Orm\Filter\AbstractFilter;
use ApiPlatform\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use ApiPlatform\Metadata\Operation;
use Doctrine\ORM\QueryBuilder;

class ActiveUserFilter extends AbstractFilter
{
    protected function filterProperty(
        string $property,
        $value,
        QueryBuilder $queryBuilder,
        QueryNameGeneratorInterface $queryNameGenerator,
        string $resourceClass,
        Operation $operation = null,
        array $context = []
    ): void {
        if ($property !== 'active') {
            return;
        }

        $alias = $queryBuilder->getRootAliases()[0];
        $queryBuilder->andWhere(sprintf('%s.active = :active', $alias))
            ->setParameter('active', filter_var($value, FILTER_VALIDATE_BOOLEAN));
    }

    public function getDescription(string $resourceClass): array
    {
        return [
            'active' => [
                'property' => 'active',
                'type' => 'bool',
                'required' => false,
            ],
        ];
    }
}

// Usage
#[ApiResource]
#[ApiFilter(ActiveUserFilter::class)]
class User
{
    // ...
}
```

## Validation

### Validation Groups
```php
use Symfony\Component\Validator\Constraints as Assert;

#[ApiResource(
    operations: [
        new Post(validationContext: ['groups' => ['Default', 'user:create']]),
        new Put(validationContext: ['groups' => ['Default', 'user:update']]),
    ]
)]
class User
{
    #[Assert\NotBlank(groups: ['user:create'])]
    #[Assert\Email]
    private string $email;

    #[Assert\NotBlank(groups: ['user:create'])]
    #[Assert\Length(min: 8, groups: ['user:create'])]
    private ?string $password = null;

    #[Assert\NotBlank(groups: ['user:update'])]
    private ?string $name = null;
}
```

## Security

### Access Control
```php
use ApiPlatform\Metadata\ApiResource;

#[ApiResource(
    operations: [
        new GetCollection(security: "is_granted('ROLE_USER')"),
        new Get(security: "is_granted('ROLE_USER')"),
        new Post(security: "is_granted('ROLE_ADMIN')"),
        new Put(security: "is_granted('ROLE_ADMIN') or object.owner == user"),
        new Delete(security: "is_granted('ROLE_ADMIN')"),
    ]
)]
class User
{
    // ...
}
```

### Security Messages
```php
#[ApiResource(
    operations: [
        new Put(
            security: "is_granted('ROLE_ADMIN') or object.owner == user",
            securityMessage: "Only admins or the owner can update this resource."
        ),
    ]
)]
class Post
{
    // ...
}
```

### JWT Authentication
```php
// config/packages/security.yaml
security:
    firewalls:
        api:
            pattern: ^/api/
            stateless: true
            jwt: ~

// Login endpoint
#[ApiResource(
    operations: [
        new Post(
            uriTemplate: '/auth/login',
            processor: LoginProcessor::class
        )
    ]
)]
class LoginRequest
{
    public string $username;
    public string $password;
}

class LoginProcessor implements ProcessorInterface
{
    public function process($data, Operation $operation, array $uriVariables = [], array $context = []): array
    {
        // Validate credentials and return JWT
        $token = $this->jwtManager->create($user);

        return [
            'token' => $token,
            'refresh_token' => $refreshToken,
        ];
    }
}
```

## Pagination

### Custom Pagination
```php
#[ApiResource(
    paginationEnabled: true,
    paginationItemsPerPage: 30,
    paginationMaximumItemsPerPage: 100,
    paginationClientEnabled: true,
    paginationClientItemsPerPage: true
)]
class User
{
    // ...
}

// Usage:
// GET /api/users?page=2
// GET /api/users?itemsPerPage=50
// GET /api/users?pagination=false  (disable pagination)
```

### Cursor-Based Pagination
```php
use ApiPlatform\State\Pagination\Pagination;

#[ApiResource(
    paginationType: 'cursor'
)]
class User
{
    // ...
}
```

## Serialization Context

### Dynamic Groups
```php
use ApiPlatform\Serializer\SerializerContextBuilderInterface;

class UserContextBuilder implements SerializerContextBuilderInterface
{
    public function __construct(
        private SerializerContextBuilderInterface $decorated
    ) {}

    public function createFromRequest(Request $request, bool $normalization, array $extractedAttributes = null): array
    {
        $context = $this->decorated->createFromRequest($request, $normalization, $extractedAttributes);

        if ($normalization && $this->security->isGranted('ROLE_ADMIN')) {
            $context['groups'][] = 'admin:read';
        }

        return $context;
    }
}
```

## OpenAPI Documentation

### Custom OpenAPI Attributes
```php
use OpenApi\Attributes as OA;

#[ApiResource(
    operations: [
        new Get(
            openapi: new OA\Operation(
                summary: 'Retrieve a user',
                description: 'Retrieves a User resource by ID',
                responses: [
                    new OA\Response(
                        response: 200,
                        description: 'User resource',
                    ),
                    new OA\Response(
                        response: 404,
                        description: 'User not found',
                    ),
                ]
            )
        ),
    ]
)]
class User
{
    // ...
}
```

## Error Handling

### Custom Exception Handler
```php
use ApiPlatform\Symfony\EventListener\ErrorListener;

class CustomErrorListener
{
    public function onKernelException(ExceptionEvent $event): void
    {
        $exception = $event->getThrowable();

        if ($exception instanceof CustomException) {
            $response = new JsonResponse([
                'error' => $exception->getMessage(),
                'code' => $exception->getCode(),
            ], $exception->getStatusCode());

            $event->setResponse($response);
        }
    }
}
```

### Validation Error Format
```php
// Automatically returns:
{
    "@context": "/api/contexts/ConstraintViolationList",
    "@type": "ConstraintViolationList",
    "hydra:title": "An error occurred",
    "hydra:description": "email: This value is not a valid email address.",
    "violations": [
        {
            "propertyPath": "email",
            "message": "This value is not a valid email address.",
            "code": "bd79c0ab-ddba-46cc-a703-a7a4b08de310"
        }
    ]
}
```

## GraphQL Support

### Enable GraphQL
```php
#[ApiResource(
    graphQlOperations: [
        new Query(),
        new QueryCollection(),
        new Mutation(name: 'create'),
        new Mutation(name: 'update'),
        new DeleteMutation(name: 'delete'),
    ]
)]
class User
{
    // ...
}

// GraphQL query:
// query {
//   user(id: "/api/users/1") {
//     email
//     name
//     posts {
//       edges {
//         node {
//           title
//         }
//       }
//     }
//   }
// }
```

## File Upload

### File Upload Handling
```php
use Symfony\Component\HttpFoundation\File\File;
use Vich\UploaderBundle\Mapping\Annotation as Vich;

#[ApiResource(
    operations: [
        new Post(
            inputFormats: ['multipart' => ['multipart/form-data']],
            processor: MediaObjectProcessor::class
        )
    ]
)]
#[Vich\Uploadable]
class MediaObject
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[Vich\UploadableField(mapping: 'media_object', fileNameProperty: 'filePath')]
    private ?File $file = null;

    #[ORM\Column(nullable: true)]
    private ?string $filePath = null;
}
```

## Versioning

### URI Versioning
```php
#[ApiResource(
    operations: [
        new Get(uriTemplate: '/v1/users/{id}'),
        new Get(uriTemplate: '/v2/users/{id}', output: UserV2Output::class),
    ]
)]
class User
{
    // ...
}
```

### Custom Header Versioning
```php
class VersionListener
{
    public function onKernelRequest(RequestEvent $event): void
    {
        $request = $event->getRequest();
        $version = $request->headers->get('API-Version', 'v1');

        $request->attributes->set('_api_version', $version);
    }
}
```

## Testing

### API Test
```php
use ApiPlatform\Symfony\Bundle\Test\ApiTestCase;

class UserTest extends ApiTestCase
{
    public function testCreateUser(): void
    {
        $response = static::createClient()->request('POST', '/api/users', [
            'json' => [
                'email' => 'test@example.com',
                'password' => 'password123',
                'name' => 'Test User',
            ],
        ]);

        $this->assertResponseStatusCodeSame(201);
        $this->assertResponseHeaderSame('content-type', 'application/ld+json; charset=utf-8');
        $this->assertJsonContains([
            '@context' => '/api/contexts/User',
            '@type' => 'User',
            'email' => 'test@example.com',
        ]);
    }

    public function testGetUser(): void
    {
        $client = static::createClient();
        $iri = $this->findIriBy(User::class, ['email' => 'test@example.com']);

        $client->request('GET', $iri);

        $this->assertResponseIsSuccessful();
        $this->assertMatchesResourceItemJsonSchema(User::class);
    }
}
```

## Performance Best Practices

1. **Use pagination** - Always enable and configure properly
2. **Optimize serialization** - Use groups to limit exposed fields
3. **Index database columns** - Used in filters and searches
4. **Cache responses** - HTTP caching and API Platform cache
5. **Use eager loading** - Prevent N+1 queries with joins
6. **Implement state providers** - For custom caching logic
7. **Use partial responses** - Allow clients to select fields

## Security Best Practices

1. **Always validate input** - Use Symfony validators
2. **Implement proper authentication** - JWT, OAuth2, etc.
3. **Use security expressions** - Control access per operation
4. **Validate file uploads** - Check MIME types and file size
5. **Rate limiting** - Prevent API abuse
6. **CORS configuration** - Properly configure allowed origins
7. **Audit logging** - Log all write operations

## Common Patterns

### Soft Delete
```php
#[ApiResource(
    operations: [
        new GetCollection(provider: SoftDeletedUserProvider::class),
        new Delete(processor: SoftDeleteProcessor::class),
    ]
)]
#[ORM\Entity]
class User
{
    #[ORM\Column(nullable: true)]
    private ?\DateTimeImmutable $deletedAt = null;
}
```

### Event Sourcing
```php
class UserEventProcessor implements ProcessorInterface
{
    public function process($data, Operation $operation, array $uriVariables = [], array $context = []): User
    {
        $event = new UserEvent($data, $operation->getName());
        $this->eventStore->append($event);

        // Apply event to aggregate
        $user = $this->applyEvent($event);

        return $user;
    }
}
```

## When Helping Users

1. **Ask about API Platform version** - Features differ between 3.x and 4.x
2. **Recommend state providers/processors** - Over direct entity exposure
3. **Suggest DTOs** - For input/output separation
4. **Emphasize validation** - Both input and business logic
5. **Implement proper security** - Per-operation access control
6. **Use filters wisely** - Both built-in and custom
7. **Document endpoints** - OpenAPI attributes and descriptions
8. **Test thoroughly** - Use ApiTestCase for comprehensive testing

Your goal is to help users build robust, secure, and performant REST APIs using API Platform's latest features and best practices.
