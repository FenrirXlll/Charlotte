# Sistema de Diseño para Charlotte ARCS Online Store

## Enfoque de Implementación

Después de analizar los requisitos y puntos difíciles del proyecto, adoptaremos un enfoque de arquitectura de microservicios para la tienda en línea de Charlotte ARCS. Este enfoque nos permitirá utilizar las diferentes tecnologías requeridas (PHP/Laravel, Java/Spring Boot, Python/Flask) de manera eficiente, asignando a cada tecnología las tareas para las que está mejor adaptada.

### Puntos Difíciles y Soluciones

1. **Integración de Múltiples Tecnologías**: 
   - Utilizaremos una API Gateway para gestionar la comunicación entre los diferentes servicios backend.
   - Implementaremos un patrón de comunicación basado en REST para mantener la consistencia.

2. **Sistema de Dropshipping Multi-proveedor**:
   - Desarrollaremos adaptadores específicos para cada API de proveedor (Amazon, Palacio de Hierro, CNFans, etc.).
   - Implementaremos un servicio de sincronización para mantener el inventario actualizado.

3. **Sistema de Pagos Dual**:
   - Integraremos Stripe para pagos en línea.
   - Desarrollaremos un sistema de seguimiento para transferencias manuales a Banco Azteca.

4. **Animaciones y Rendimiento**:
   - Optimizaremos la carga de las animaciones Sakura para evitar problemas de rendimiento.
   - Implementaremos lazy loading para cargar recursos visuales pesados solo cuando sean necesarios.

5. **Seguridad**:
   - Implementaremos HTTPS obligatorio.
   - Utilizaremos tokens JWT para autenticación de APIs.
   - Implementaremos validación CSRF en formularios.
   - Securizaremos las conexiones a Supabase.

### Frameworks y Bibliotecas

- **Frontend**:
  - Bootstrap 5.3.3 para responsive design
  - Chart.js 4.4.2 para visualizaciones
  - Sakura para animaciones de pétalos
  - YouTube IFrame Player API para reproducción de videos

- **Backend**:
  - Laravel 11 para rutas y autenticación
  - Spring Boot 3.2 para APIs robustas
  - Flask 3.0 para microservicios ligeros
  - Supabase para almacenamiento y gestión de datos

## Arquitectura del Sistema

### Arquitectura General

Implementaremos una arquitectura de microservicios con los siguientes componentes principales:

1. **Frontend (Cliente Web)**:
   - Interfaz de usuario basada en HTML5, CSS3 y JavaScript.
   - Responsable de la presentación y la experiencia del usuario.

2. **API Gateway (Laravel)**:
   - Punto de entrada principal para todas las solicitudes.
   - Gestiona la autenticación, enrutamiento y balanceo de carga.

3. **Servicios de Productos (Java/Spring Boot)**:
   - Gestión del catálogo de productos.
   - Integración con APIs de dropshipping.
   - Actualización de inventario en tiempo real.

4. **Servicios de Usuarios y Votaciones (PHP/Laravel)**:
   - Gestión de usuarios, autenticación y sesiones.
   - Sistema de votación para artistas.
   - Gestión de membresías.

5. **Servicios de Notificaciones (Python/Flask)**:
   - Envío de correos electrónicos.
   - Gestión de postulaciones.
   - Notificaciones del sistema.

6. **Base de Datos (Supabase)**:
   - Almacenamiento persistente de datos del sistema.
   - Tablas para productos, artistas, votos, etc.

7. **Procesador de Pagos**:
   - Integración con Stripe.
   - Sistema de registro y verificación de transferencias bancarias.

### Flujo de Datos

El flujo de datos a través del sistema seguirá generalmente este patrón:

1. El usuario interactúa con la interfaz web.
2. Las solicitudes son enviadas al API Gateway.
3. El API Gateway enruta la solicitud al servicio apropiado.
4. El servicio procesa la solicitud y consulta/actualiza la base de datos según sea necesario.
5. La respuesta sigue el camino inverso hacia el usuario.

## Estructuras de Datos e Interfaces

Se han definido las estructuras de datos y clases principales en un diagrama de clases mermaid en el archivo `charlotte_arcs_class_diagram.mermaid`. Estas clases representan las entidades principales del sistema y sus relaciones.

### Descripción de las Clases Principales

1. **User**: Representa a un usuario registrado en el sistema. Incluye información de perfil, tipo de membresía y métodos para autenticación.

2. **Product**: Representa un producto en la tienda, con atributos como nombre, precio, categoría y proveedor. Incluye métodos para actualizar stock y obtener detalles.

3. **Cart y CartItem**: Representan el carrito de compras y sus elementos.

4. **Order y OrderItem**: Representan una orden de compra y sus elementos.

5. **PaymentProcessor**: Gestiona los pagos mediante Stripe y las transferencias manuales a Banco Azteca.

6. **Artist**: Representa a un artista en la plataforma, con información sobre sus canciones, géneros y votos.

7. **Vote**: Representa un voto de un usuario a un artista específico.

8. **Nomination**: Representa una postulación para nuevos artistas.

9. **ProductService**: Gestiona las operaciones relacionadas con productos, incluyendo la sincronización con proveedores externos.

10. **ProviderAdapter y sus implementaciones**: Define la interfaz para los adaptadores de las diferentes APIs de proveedores de dropshipping.

11. **ArtistService**: Gestiona las operaciones relacionadas con los artistas y el sistema de votación.

12. **NominationService**: Gestiona las postulaciones de nuevos artistas.

13. **EmailService**: Gestiona el envío de correos electrónicos.

14. **SupabaseClient**: Proporciona métodos para interactuar con la base de datos Supabase.

15. **AuthService**: Gestiona la autenticación de usuarios.

16. **MembershipService**: Gestiona los diferentes tipos de membresía y sus beneficios.

## Flujo de Control del Programa

Se ha definido un diagrama de secuencia completo en el archivo `charlotte_arcs_sequence_diagram.mermaid`. Este diagrama muestra el flujo de operaciones para los siguientes escenarios principales:

1. **Carga inicial de la página**
2. **Registro de usuario**
3. **Navegación y búsqueda de productos**
4. **Agregar productos al carrito**
5. **Proceso de checkout y pago**
6. **Votación de artistas**
7. **Postulación de artistas**
8. **Actualización de membresía**
9. **Inicialización del sistema**

Cada uno de estos flujos muestra cómo interactúan los diferentes componentes del sistema para proporcionar la funcionalidad requerida.

## Implementación de APIs

### APIs para el Sistema de Dropshipping

#### ProductService API (Java/Spring Boot)

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    @Autowired
    private ProductService productService;
    
    @GetMapping
    public ResponseEntity<List<Product>> getProducts(
        @RequestParam(required = false) String category,
        @RequestParam(required = false) String filter,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size) {
        return ResponseEntity.ok(productService.getProducts(category, filter, page, size));
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable String id) {
        return ResponseEntity.ok(productService.getProductById(id));
    }
    
    @GetMapping("/sync")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<String> syncInventory() {
        productService.syncInventory();
        return ResponseEntity.ok("Inventory synchronization initiated");
    }
}
```

#### ProviderAdapter implementación (ejemplo para Amazon)

```java
@Service
public class AmazonAdapter implements ProviderAdapter {
    private final String API_KEY = "your-amazon-api-key";
    private final String API_URL = "https://www.amazon.com.mx/api/remates";
    
    @Override
    public List<Product> fetchProducts(String query) {
        // Implementación para consumir la API de Amazon
        // y transformar los resultados al formato interno
        return amazonProducts.stream()
            .map(this::mapToProduct)
            .collect(Collectors.toList());
    }
    
    @Override
    public boolean checkAvailability(String productId) {
        // Verificar disponibilidad en Amazon
        return true; // Simplificado para el ejemplo
    }
    
    @Override
    public float getPrice(String productId) {
        // Obtener precio actualizado desde Amazon
        // Aplicar margen del 15% requerido
        float basePrice = 100.0f; // Ejemplo
        return basePrice * 1.15f;
    }
    
    private Product mapToProduct(AmazonProduct amazonProduct) {
        Product product = new Product();
        product.setId(amazonProduct.getId());
        product.setName(amazonProduct.getTitle());
        product.setPrice(amazonProduct.getPrice() * 1.15f); // 15% de margen
        product.setCategory(mapCategory(amazonProduct.getCategory()));
        product.setImages(amazonProduct.getImageUrls());
        product.setDescription(amazonProduct.getDescription());
        product.setStock(amazonProduct.getStock());
        product.setProvider("Amazon");
        product.setProviderProductId(amazonProduct.getId());
        product.setActive(true);
        return product;
    }
    
    private String mapCategory(String amazonCategory) {
        // Mapear categorías de Amazon a categorías internas
        switch (amazonCategory.toLowerCase()) {
            case "men's clothing":
                return "hombre";
            case "women's clothing":
                return "mujer";
            // más mapeos...
            default:
                return "otros";
        }
    }
}
```

### APIs para el Sistema de Votación de Artistas

#### ArtistController (PHP/Laravel)

```php
<?php

namespace App\Http\Controllers;

use App\Models\Artist;
use App\Models\Vote;
use App\Services\MembershipService;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class ArtistController extends Controller
{
    protected $membershipService;
    
    public function __construct(MembershipService $membershipService)
    {
        $this->membershipService = $membershipService;
    }
    
    public function index()
    {
        $artists = Artist::all();
        $votingStats = [];
        
        foreach ($artists as $artist) {
            $votingStats[$artist->id] = [
                'votes' => Vote::where('artist_id', $artist->id)->count(),
                'percentage' => $this->calculatePercentage($artist->id)
            ];
        }
        
        return response()->json([
            'artists' => $artists,
            'stats' => $votingStats
        ]);
    }
    
    public function show($id)
    {
        $artist = Artist::findOrFail($id);
        $songs = $artist->songs;
        $upcoming = $artist->upcomingSongs;
        
        return response()->json([
            'artist' => $artist,
            'songs' => $songs,
            'upcoming' => $upcoming,
            'votes' => Vote::where('artist_id', $id)->count()
        ]);
    }
    
    public function vote(Request $request)
    {
        $request->validate([
            'artist_id' => 'required|exists:artists,id'
        ]);
        
        $user = Auth::user();
        
        if (!$user) {
            return response()->json(['error' => 'Unauthorized'], 401);
        }
        
        // Verificar membresía
        $membership = $this->membershipService->getUserMembership($user->id);
        
        if (!$membership) {
            return response()->json([
                'error' => 'Membership required',
                'membership_options' => [
                    'basic' => [
                        'price_monthly' => 5,
                        'price_yearly' => 55
                    ],
                    'vip' => [
                        'price_monthly' => 20,
                        'price_yearly' => 200
                    ],
                    'premium' => [
                        'price_monthly' => 155,
                        'price_yearly' => 1600
                    ]
                ]
            ], 403);
        }
        
        // Crear voto
        $vote = new Vote();
        $vote->user_id = $user->id;
        $vote->artist_id = $request->artist_id;
        $vote->timestamp = now();
        $vote->save();
        
        // Actualizar contador de votos del artista
        $artist = Artist::find($request->artist_id);
        $artist->votes = Vote::where('artist_id', $request->artist_id)->count();
        $artist->save();
        
        return response()->json(['success' => true]);
    }
    
    private function calculatePercentage($artistId)
    {
        $artistVotes = Vote::where('artist_id', $artistId)->count();
        $totalVotes = Vote::count();
        
        if ($totalVotes === 0) {
            return 0;
        }
        
        return ($artistVotes / $totalVotes) * 100;
    }
}
```

### APIs para las Postulaciones

#### NominationController (Python/Flask)

```python
from flask import Blueprint, request, jsonify
from supabase_client import supabase_client
from email_service import send_nomination_confirmation, send_nomination_notification
import uuid
from datetime import datetime

nomination_bp = Blueprint('nomination', __name__)

@nomination_bp.route('/api/nominations', methods=['POST'])
def submit_nomination():
    data = request.get_json()
    
    # Validar campos requeridos
    required_fields = ['name', 'email', 'portfolio', 'description']
    for field in required_fields:
        if field not in data:
            return jsonify({'error': f'Field {field} is required'}), 400
    
    # Crear ID único para la nominación
    nomination_id = str(uuid.uuid4())
    
    # Preparar datos para insertar en Supabase
    nomination_data = {
        'id': nomination_id,
        'name': data['name'],
        'email': data['email'],
        'portfolio': data['portfolio'],
        'description': data['description'],
        'reviewed': False,
        'created_at': datetime.now().isoformat()
    }
    
    # Insertar en Supabase
    result = supabase_client.table('nominations').insert(nomination_data).execute()
    
    # Verificar error
    if 'error' in result:
        return jsonify({'error': 'Failed to save nomination'}), 500
    
    # Enviar correos
    send_nomination_confirmation(data['email'], data['name'])
    send_nomination_notification('atencionactech024@gmail.com', nomination_data)
    
    return jsonify({
        'success': True,
        'nomination_id': nomination_id,
        'message': 'Your nomination has been submitted successfully.'
    }), 201
    
@nomination_bp.route('/api/nominations', methods=['GET'])
def get_nominations():
    # Verificar autenticación administrativa (simplificado para el ejemplo)
    # En producción usar un sistema de autenticación completo
    
    # Obtener nominaciones desde Supabase
    result = supabase_client.table('nominations').select('*').execute()
    
    if 'error' in result:
        return jsonify({'error': 'Failed to retrieve nominations'}), 500
        
    return jsonify({'nominations': result['data']}), 200
```

### Integración de Pagos

#### PaymentController (PHP/Laravel)

```php
<?php

namespace App\Http\Controllers;

use App\Models\Order;
use App\Models\User;
use App\Services\PaymentService;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Stripe\Stripe;
use Stripe\PaymentIntent;

class PaymentController extends Controller
{
    protected $paymentService;
    
    public function __construct(PaymentService $paymentService)
    {
        $this->paymentService = $paymentService;
    }
    
    public function processStripePayment(Request $request)
    {
        $request->validate([
            'amount' => 'required|numeric',
            'order_id' => 'required',
            'payment_method_id' => 'required',
        ]);
        
        try {
            Stripe::setApiKey(config('services.stripe.secret'));
            
            // Crear PaymentIntent
            $paymentIntent = PaymentIntent::create([
                'amount' => $request->amount * 100, // Convertir a centavos
                'currency' => 'mxn',
                'payment_method' => $request->payment_method_id,
                'confirmation_method' => 'manual',
                'confirm' => true,
                'metadata' => [
                    'order_id' => $request->order_id,
                    'user_id' => Auth::id()
                ],
            ]);
            
            // Actualizar el estado del pedido
            $order = Order::findOrFail($request->order_id);
            $order->payment_status = 'paid';
            $order->payment_id = $paymentIntent->id;
            $order->save();
            
            return response()->json([
                'success' => true,
                'payment_intent' => $paymentIntent
            ]);
        } catch (\Exception $e) {
            return response()->json([
                'error' => $e->getMessage()
            ], 500);
        }
    }
    
    public function processManualTransfer(Request $request)
    {
        $request->validate([
            'amount' => 'required|numeric',
            'order_id' => 'required',
            'transfer_reference' => 'required|string',
        ]);
        
        try {
            // Registrar la transferencia manual
            $transferReference = $this->paymentService->registerManualTransfer(
                $request->order_id,
                $request->amount,
                $request->transfer_reference
            );
            
            // Actualizar el estado del pedido
            $order = Order::findOrFail($request->order_id);
            $order->payment_status = 'pending_verification';
            $order->payment_reference = $request->transfer_reference;
            $order->save();
            
            return response()->json([
                'success' => true,
                'message' => 'Su transferencia ha sido registrada y está pendiente de verificación',
                'reference' => $transferReference
            ]);
        } catch (\Exception $e) {
            return response()->json([
                'error' => $e->getMessage()
            ], 500);
        }
    }
    
    public function getPaymentMethods()
    {
        return response()->json([
            'stripe' => [
                'public_key' => config('services.stripe.key'),
                'supported_cards' => ['visa', 'mastercard']
            ],
            'bank_transfer' => [
                'bank_name' => 'Banco Azteca',
                'account_holder' => 'Charlotte ARCS',
                'clabe' => '127180016932198547',
                'instructions' => 'Realice su transferencia y guarde su comprobante para verificación'
            ]
        ]);
    }
}
```

## Seguridad

### Configuración de Seguridad

1. **HTTPS Obligatorio**:
   - Implementaremos redirección forzada de HTTP a HTTPS en el servidor web.
   - Configuraremos certificados SSL/TLS válidos para el dominio.

2. **Autenticación y Autorización**:
   - Utilizaremos tokens JWT para autenticación de APIs.
   - Implementaremos roles de usuario (cliente, miembro, administrador).
   - Protegeremos rutas sensibles con middleware de autenticación.

3. **Protección contra Ataques**:
   - CSRF: Implementaremos tokens en todos los formularios.
   - XSS: Aplicaremos sanitización de entrada y salida de datos.
   - SQL Injection: Utilizaremos consultas parametrizadas a través de ORM.
   - CORS: Configuraremos políticas de origen adecuadas.

4. **Seguridad en Pagos**:
   - Stripe: Utilizaremos Elements para minimizar la exposición a datos sensibles.
   - No almacenaremos datos de tarjetas en nuestra base de datos.
   - Banco Azteca: Implementaremos validación manual de transferencias.

5. **Seguridad de la Base de Datos**:
   - Supabase: Configuraremos políticas de Row Level Security (RLS).
   - Restringiremos acceso directo a tablas sensibles.
   - Encriptaremos datos sensibles como contraseñas.

### Ejemplo de Configuración de Supabase RLS

```sql
-- Política para productos (acceso de lectura público)
CREATE POLICY "Productos visibles para todos" ON products FOR SELECT USING (true);

-- Política para carrito (solo el propietario puede ver y modificar)
CREATE POLICY "Carrito accesible solo por el propietario" ON cart_items 
  FOR ALL USING (auth.uid() = user_id);

-- Política para órdenes (solo el propietario y los administradores)
CREATE POLICY "Ordenes visibles para el propietario" ON orders 
  FOR SELECT USING (auth.uid() = user_id);
  
CREATE POLICY "Ordenes administrables por admins" ON orders 
  FOR ALL USING (auth.role() = 'admin');

-- Política para votos (solo miembros pueden votar)
CREATE POLICY "Votar requiere membresía" ON votes 
  FOR INSERT WITH CHECK (auth.uid() IN (
    SELECT user_id FROM memberships WHERE expiration_date > NOW()
  ));
```

## Puntos no claros y consideraciones adicionales

1. **Sincronización con Múltiples Proveedores**: La integración con diferentes APIs de dropshipping puede ser compleja debido a las diferencias en formatos y especificaciones. Recomendamos implementar un sistema de adaptadores flexible que pueda ajustarse a nuevos proveedores fácilmente.

2. **Conciliación de Pagos Manuales**: El proceso de verificación de transferencias bancarias requerirá un panel administrativo para que el personal pueda verificar y confirmar los pagos manualmente. Esto se desarrollará como parte del sistema pero no es visible para los clientes.

3. **Rendimiento de Animaciones**: Las animaciones Sakura y los elementos visuales complejos pueden afectar el rendimiento en dispositivos con recursos limitados. Implementaremos opciones para reducir animaciones en dispositivos móviles o conexiones lentas.

4. **Estrategia de Cacheo**: Para mejorar el rendimiento, implementaremos una estrategia de cacheo para:  
   - Catálogo de productos
   - Página de artistas
   - Resultados de votaciones

5. **Internacionalización**: Aunque el enfoque principal es México, la tienda atenderá a varios países. Implementaremos soporte para múltiples monedas, idiomas (español, inglés, portugués, estonio) y opciones de envío internacional.

6. **Escalabilidad**: La arquitectura de microservicios nos permitirá escalar horizontalmente cada componente según sea necesario. Especialmente importante para el sistema de productos durante periodos de alta demanda.

## Conclusión

La arquitectura propuesta para la tienda en línea de Charlotte ARCS satisface todos los requisitos técnicos y funcionales especificados en el PRD y las especificaciones del cliente. El enfoque de microservicios permite utilizar las tecnologías requeridas de manera eficiente, y la estructura modular facilita el mantenimiento y la escalabilidad del sistema.

La implementación seguirá la estructura de carpetas y archivos especificada, y todas las funcionalidades clave como el sistema de dropshipping, votación de artistas, postulaciones, y membresías se desarrollarán según lo requerido. La seguridad y el rendimiento se han considerado como aspectos fundamentales del diseño.

Con esta arquitectura, Charlotte ARCS contará con una plataforma robusta y flexible que fusiona moda, tecnología y arte de acuerdo con la visión de Carla Sánchez, alineándose perfectamente con los valores de sostenibilidad, creatividad y compasión de la marca.
