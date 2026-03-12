# FIELD HERO: La Ruta del Técnico
## Juego educativo de plataformas 2D pixel art — Motor: Phaser 3

---

## DESCRIPCIÓN DEL PROYECTO

Juego de plataformas 2D pixel art como herramienta de formación interactiva para técnicos de campo (Service Representatives) de Schneider Electric. Enseña el programa **F2O (Field to Order)**: cómo detectar oportunidades de negocio (leads) durante visitas de servicio, distinguir leads válidos de inválidos, y entender el sistema de incentivos.

**Público objetivo:** Técnicos de 20-27 años, recién incorporados.
**Fecha de uso:** 20 de marzo de 2026.

---

## STACK TÉCNICO

- **Phaser 3** (v3.90.0) como motor de juego — cargado desde CDN de jsDelivr
- **UN SOLO ARCHIVO index.html** con todo el código del juego en un `<script>` inline
- **JavaScript vanilla** dentro del HTML — sin TypeScript, sin bundlers, sin npm
- **Sprites generados por código** usando la API gráfica de Phaser (`this.add.graphics()`, `this.textures.createCanvas()`, o `generateTexture()`) — SIN imágenes externas
- **Phaser Arcade Physics** para gravedad, colisiones y movimiento del personaje
- **Phaser Scenes** para gestionar las pantallas del juego (título, briefing, nivel, quiz, resumen)
- Para ejecutar: servidor local simple (`python -m http.server 8080` o Live Server de VS Code)

### CDN de Phaser (incluir en el HTML):
```html
<script src="https://cdn.jsdelivr.net/npm/phaser@3.90.0/dist/phaser.min.js"></script>
```

### Alternativa offline:
Descargar phaser.min.js al directorio del proyecto y referenciar con:
```html
<script src="phaser.min.js"></script>
```

---

## PALETA DE COLORES (valores hex para Phaser — sin el #, formato 0xRRGGBB)

```
Verde SE principal:    0x3DCD58  (#3DCD58)
Verde SE oscuro:       0x2a9e40  (#2a9e40)
Verde SE claro:        0x6de882  (#6de882)
Negro fondo:           0x1a1a2e  (#1a1a2e)
Gris oscuro:           0x2d2d44  (#2d2d44)
Gris medio:            0x4a4a5e  (#4a4a5e)
Gris claro:            0x7a7a8e  (#7a7a8e)
Blanco:                0xe8e8f0  (#e8e8f0)
Rojo alerta:           0xe84057  (#e84057)
Rojo oscuro:           0xb82838  (#b82838)
Naranja chaleco:       0xf0a030  (#f0a030)
Amarillo peligro:      0xf0d020  (#f0d020)
Azul pantalón:         0x4080d0  (#4080d0)
Azul oscuro:           0x2a5090  (#2a5090)
Piel:                  0xf0c8a0  (#f0c8a0)
Piel oscuro:           0xd0a878  (#d0a878)
Marrón botas:          0x8b6040  (#8b6040)
Metal oscuro:          0x3a4a5a  (#3a4a5a)
Metal medio:           0x5a6a7a  (#5a6a7a)
Metal claro:           0x8a9aaa  (#8a9aaa)
Suelo oscuro:          0x3a3a4a  (#3a3a4a)
Suelo medio:           0x4a4a5a  (#4a4a5a)
Pared oscura:          0x2a2a3a  (#2a2a3a)
Cable azul:            0x2040a0  (#2040a0)
Cable rojo:            0xc03030  (#c03030)
Cable amarillo:        0xc0b020  (#c0b020)
```

---

## ESTRUCTURA DE SCENES DE PHASER

```
BootScene        → Genera todas las texturas/sprites por código (no carga archivos)
TitleScene       → Pantalla de título con "Pulsa ENTER para empezar"
BriefingScene    → Briefing del nivel actual antes de jugar
GameScene        → El nivel jugable de plataformas (Mundo 1, 2 o 3)
QuizScene        → Overlay de pregunta al inspeccionar un equipo (se lanza sobre GameScene)
SummaryScene     → Resumen de puntuación al terminar un nivel
BossScene        → Nivel final: diálogo con el Facility Manager
FinalScene       → Pantalla final con puntuación total y perfil del técnico
```

### Configuración Phaser recomendada:
```javascript
const config = {
    type: Phaser.AUTO,
    width: 800,
    height: 450,
    parent: 'game',
    backgroundColor: '#1a1a2e',
    physics: {
        default: 'arcade',
        arcade: {
            gravity: { y: 800 },
            debug: false
        }
    },
    scale: {
        mode: Phaser.Scale.FIT,
        autoCenter: Phaser.Scale.CENTER_BOTH
    },
    scene: [BootScene, TitleScene, BriefingScene, GameScene, QuizScene, SummaryScene]
};
```

---

## GENERACIÓN DE SPRITES POR CÓDIGO (BootScene)

Como NO usamos imágenes externas, todos los sprites se generan en la BootScene usando la API gráfica de Phaser. El patrón es:

```javascript
// Ejemplo: crear una textura de sprite por código
const graphics = this.add.graphics();
graphics.fillStyle(0x3DCD58);
graphics.fillRect(0, 0, 32, 32);
graphics.generateTexture('miSprite', 32, 32);
graphics.destroy();
```

Para sprites con detalle pixel art, usar `this.textures.createCanvas()` y dibujar píxel a píxel:
```javascript
const canvas = this.textures.createCanvas('technico', 32, 32);
const ctx = canvas.getContext();
// Dibujar píxel a píxel con ctx.fillStyle y ctx.fillRect(x, y, 1, 1)
canvas.refresh();
```

### Sprites a generar en BootScene:

**Personaje — Técnico SR (32x32px):**
- Casco verde SE (#3DCD58) en la cabeza
- Cara color piel (#f0c8a0) con ojos negros
- Chaleco naranja reflectante (#f0a030) con franjas verdes SE
- Guantes grises (#4a4a5e) en las manos
- Pantalón azul de trabajo (#4080d0)
- Botas marrones de seguridad (#8b6040)
- Generar al menos 2 frames: 'tech_idle' y 'tech_walk1', 'tech_walk2' para animación

**Leads válidos — Bombilla verde (16x16px):**
- Forma de bombilla redondeada, color verde claro (#6de882) con brillo blanco
- Nombre textura: 'lead_valid'

**Leads inválidos — Esfera roja (16x16px):**
- Forma circular, color rojo (#e84057) con X blanca
- Nombre textura: 'lead_invalid'

**Tiles de entorno (32x32px cada uno):**
- 'tile_floor': baldosa industrial gris con borde sutil
- 'tile_wall': bloque de pared gris oscuro
- 'tile_platform': plataforma metálica (pasarela)
- 'tile_danger': suelo con franja amarilla/negra de peligro

**Equipos eléctricos (texturas más grandes):**
- 'equip_panel': cuadro eléctrico LV (48x64px) — gris con indicadores LED
- 'equip_ups': torre UPS (36x64px) — gris oscuro con pantalla verde
- 'equip_rack': rack de servidores (36x80px) — negro con luces parpadeo
- 'equip_transformer': transformador (64x56px) — marrón/cobre grande
- 'equip_cooling': unidad de cooling (48x48px) — gris con ventilador

**Otros:**
- 'hazard_water': charco de agua (48x12px) — azul semitransparente
- 'epi_gloves': guantes protección (16x16px)
- 'epi_boots': botas aislantes (16x16px)
- 'danger_sign': señal de peligro triangular (24x24px)

---

## MECÁNICAS DE JUEGO (implementación Phaser)

### Personaje (Phaser Arcade Physics Sprite):
```javascript
this.player = this.physics.add.sprite(100, 300, 'tech_idle');
this.player.setCollideWorldBounds(true);
this.player.setBounce(0.1);
this.player.body.setSize(20, 30); // hitbox más pequeña que el sprite
```

### Controles (Phaser Cursors + teclas adicionales):
```javascript
this.cursors = this.input.keyboard.createCursorKeys();
this.keyA = this.input.keyboard.addKey('A');
this.keyD = this.input.keyboard.addKey('D');
this.keyW = this.input.keyboard.addKey('W');
this.keyE = this.input.keyboard.addKey('E');
this.keyESC = this.input.keyboard.addKey('ESC');
```

### Movimiento en update():
```javascript
if (this.cursors.left.isDown || this.keyA.isDown) {
    this.player.setVelocityX(-200);
    this.player.setFlipX(true);
    this.player.anims.play('walk', true);
} else if (this.cursors.right.isDown || this.keyD.isDown) {
    this.player.setVelocityX(200);
    this.player.setFlipX(false);
    this.player.anims.play('walk', true);
} else {
    this.player.setVelocityX(0);
    this.player.anims.play('idle', true);
}

if ((this.cursors.up.isDown || this.keyW.isDown || this.cursors.space.isDown) 
    && this.player.body.touching.down) {
    this.player.setVelocityY(-450);
}
```

### Nivel como Tilemap estático:
Usar `this.physics.add.staticGroup()` para plataformas y suelo. El nivel es más ancho que la pantalla (ej: 2400px) con cámara que sigue al jugador:
```javascript
this.cameras.main.setBounds(0, 0, 2400, 450);
this.cameras.main.startFollow(this.player, true, 0.1, 0.1);
```

### Leads como sprites con tween de flotación:
```javascript
const lead = this.physics.add.sprite(x, y, 'lead_valid');
lead.body.allowGravity = false;
this.tweens.add({
    targets: lead,
    y: y - 8,
    duration: 1000,
    yoyo: true,
    repeat: -1,
    ease: 'Sine.easeInOut'
});
```

### Detección de proximidad para inspección:
En update(), comprobar distancia entre player y cada lead:
```javascript
const dist = Phaser.Math.Distance.Between(this.player.x, this.player.y, lead.x, lead.y);
if (dist < 50) {
    this.promptText.setVisible(true); // Mostrar "[E] Inspeccionar"
    if (Phaser.Input.Keyboard.JustDown(this.keyE)) {
        this.launchQuiz(lead.quizData);
    }
}
```

### Quiz como Scene overlay:
```javascript
this.scene.pause('GameScene');
this.scene.launch('QuizScene', { quizData: quizData, callback: this.onQuizComplete.bind(this) });
```

### Peligros (charcos de agua):
```javascript
this.physics.add.overlap(this.player, this.hazards, (player, hazard) => {
    this.health -= 0.5; // Daño continuo
    this.updateHealthBar();
});
```

---

## CONTENIDO FORMATIVO (datos reales del programa F2O Iberia)

### ¿Qué es un Lead?
Un Lead (también llamado Opportunity Notification u ON) es una potencial oportunidad que puede resultar en un pedido. Es un record type en bFS usado por SRs, PMs y CSS para enviar una oportunidad. Todo el cálculo de incentivos se basa en las Opp Notif enviadas.

### Leads VÁLIDOS:
- Leads originados desde una Work Order (WO)
- Peticiones de cliente directamente a los detectores (SR, PM, CSS)
- NUEVOS contratos de mantenimiento
- Oportunidades de consultoría recomendadas por detectores
- Todas las opp debajo de las Services Product Lines
- Para contratos: no por WO sino por cliente (varios equipos = un contrato)

### Leads INVÁLIDOS:
- RENOVACIONES de contratos existentes
- Leads provenientes de campañas de marketing
- Leads que ya existen en bFS
- Leads del CCC, ISSR, o customer inquiry (no del detector)
- Leads no declarados en bFS
- Cambio de componentes SIN confirmación del cliente a más de 1 año
- Equipos bajo contrato que ya incluyen las piezas

### Proceso F2O completo (6 pasos):
1. SR/PM/CSS identifica oportunidad potencial en site
2. Crea una ON en el sistema usando herramientas F2O
3. Conversion Leader / sistema automático califica la ON
4. Si válida → Sales gestiona la oportunidad
5. Si gana (Stage 7) → Se calcula el incentivo
6. El detector recibe el incentivo

### Sistema de incentivos Iberia 2025:
- España y Portugal, todas las BU
- No retroactivo, pago cuatrimestral (cobro Enero, Abril, Agosto, Octubre)
- Oportunidades (excepto contratos): 3% sobre el importe ganado, máximo 3.000€
- Contratos nuevos/ampliaciones: importe fijo por BU (115€-430€ según BU y país)
- Aplica a FSRs y PMs
- OPP debe ser detectada Y ganada a partir de Abril 2025

### Filtros de legitimidad de oportunidades:
1. Antigüedad: sin actualización real en 12 meses = caducada
2. Evidencia de contacto: si cliente no respondió o rechazó oferta anterior, pierde legitimidad
3. Contexto: si cambió scope, producto, normativa, site o comprador → oportunidad antigua pierde validez

---

## QUIZZES POR MUNDO

### Mundo 1 — Subestación LV (4 quizzes + 2 trampas):

**Quiz 1 - Cuadro con corrosión visible:**
Pregunta: "Detectas oxidación y humedad en la envolvente del cuadro LV. ¿Qué oportunidad generas?"
- A) Renovación del contrato existente → ❌ "Las renovaciones NO cuentan como lead F2O"
- B) EcoConsult + Modernización LV → ✅ "¡Correcto! La corrosión indica necesidad de evaluación y posible modernización"
- C) Cambio de baterías UPS → ❌ "Esto es un cuadro LV, no un UPS. Observa bien el equipo"

**Quiz 2 - Equipo sin contrato:**
Pregunta: "El cliente comenta que este cuadro no tiene contrato de mantenimiento. ¿Qué haces?"
- A) No es mi tema, ya se lo dirá ventas → ❌ "Como Trusted Advisor, TÚ eres quien detecta la oportunidad"
- B) Genero un lead para nuevo contrato de mantenimiento → ✅ "¡Exacto! Nuevos contratos son leads válidos F2O"
- C) Le digo que llame al CCC → ❌ "Los leads del CCC son INVÁLIDOS para F2O. Tú eres el detector"

**Quiz 3 - Relé de protección antiguo:**
Pregunta: "Ves un relé de protección de más de 15 años sin recambios disponibles. ¿Qué oportunidad detectas?"
- A) Modernización LV + Critical Spares → ✅ "¡Bien visto! Equipo obsoleto = oportunidad de modernización"
- B) Battery Replacement → ❌ "No es una batería. Identifica correctamente el equipo"
- C) Ya está registrado por el ISSR → ❌ "No asumas. Además, leads del ISSR no cuentan como F2O"

**Quiz 4 - Cliente pregunta por monitorización:**
Pregunta: "El facility manager pregunta: '¿Hay algo para ver el estado de los equipos desde mi oficina?'"
- A) Le digo que no es posible → ❌ "¡Sí es posible! Perdiste una oportunidad de ser Trusted Advisor"
- B) Le explico EcoStruxure Asset Advisor y genero un lead → ✅ "¡Perfecto! Detectas la necesidad y ofreces solución"
- C) Le paso el contacto del comercial sin más → ❌ "Puedes hacer más: explica el valor y genera tú el lead"

**Lead inválido 1:** "Renovación contrato mantenimiento anual" → "INVÁLIDO: Las renovaciones de contratos existentes no cuentan como leads F2O"
**Lead inválido 2:** "Lead de campaña marketing registrado" → "INVÁLIDO: Los leads de campañas no son originados por el detector"

### Mundo 2 — Sala IT / Data Centre (4 quizzes + 2 trampas):

**Quiz 1 - UPS con baterías viejas:**
Pregunta: "La pantalla del UPS indica baterías con 6 años de antigüedad. ¿Qué lead generas?"
- A) Battery Replacement → ✅ "¡Correcto! Baterías de más de 4-5 años son oportunidad clara"
- B) Nuevo contrato de cooling → ❌ "Cooling y baterías son cosas distintas. Observa el equipo"
- C) No hago nada, las baterías aún funcionan → ❌ "Como Trusted Advisor debes anticipar riesgos"

**Quiz 2 - Cooling sin mantenimiento:**
Pregunta: "Las unidades de cooling no tienen etiqueta de último mantenimiento. El cliente dice que nunca les han hecho revisión."
- A) Complete Cooling Audit + Maintenance Contract → ✅ "¡Exacto! Doble oportunidad: auditoría + contrato"
- B) Cybersecurity Assessment → ❌ "Cooling y ciberseguridad no están relacionados"
- C) Es problema del instalador, no nuestro → ❌ "Todo equipo en site del cliente es oportunidad para nosotros"

**Quiz 3 - Sin monitorización conectada:**
Pregunta: "El data centre no tiene ningún sistema de monitorización remota de los equipos."
- A) Connected Systems + EAA → ✅ "¡Perfecto! La conectividad es clave para data centres modernos"
- B) Power Factor Correction → ❌ "PFC no tiene relación con monitorización"
- C) Safety Training → ❌ "Training no resuelve la falta de monitorización"

**Quiz 4 - Cliente menciona ampliación:**
Pregunta: "El IT manager dice: 'El año que viene ampliamos la sala con 10 racks más.'"
- A) No es tema de servicio, es de proyectos → ❌ "¡Tú puedes generar el lead! No dejes pasar la oportunidad"
- B) UPS Modernisation + Cooling App & Modernisation → ✅ "¡Bien! Más racks = más potencia y refrigeración necesaria"
- C) Extended Warranty de los racks actuales → ❌ "La ampliación necesita capacidad nueva, no garantía de lo actual"

**Lead inválido 1:** "Equipo bajo contrato full que incluye piezas" → "INVÁLIDO: Si el contrato ya cubre las piezas, no genera nuevo lead"
**Lead inválido 2:** "Lead detectado por el CCC" → "INVÁLIDO: Los leads del CCC/ISSR no cuentan como F2O"

### Mundo 3 — Subestación HV (4 quizzes + 2 trampas):

**Quiz 1 - Celdas de MT antiguas:**
Pregunta: "Las celdas de media tensión tienen más de 25 años y el fabricante ya no da soporte."
- A) HV Modernisation → ✅ "¡Correcto! Equipo sin soporte del fabricante es prioridad de modernización"
- B) Battery Replacement → ❌ "Esto son celdas de MT, no baterías"
- C) LV Modernisation → ❌ "Es media tensión (HV), no baja tensión (LV)"

**Quiz 2 - Sin conmutación automática:**
Pregunta: "El cliente no tiene equipos de conmutación automática en caso de fallo de suministro."
- A) ASCO Power + ASCO Switching → ✅ "¡Exacto! ASCO es la solución para conmutación y transferencia automática"
- B) Harmonic Studies → ❌ "Los armónicos no resuelven la falta de redundancia"
- C) F-Gas assessment → ❌ "F-Gas es para refrigerantes, no para conmutación"

**Quiz 3 - Transformador sin monitorización:**
Pregunta: "El transformador principal no tiene sensores ni monitorización de temperatura o gases."
- A) Safety Training → ❌ "Training no añade sensores al transformador"
- B) EcoStruxure Asset Advisor + Maintenance Contract → ✅ "¡Bien! Monitorización + mantenimiento preventivo"
- C) Renovar el contrato existente → ❌ "Renovaciones no cuentan como F2O"

**Quiz 4 - Personal sin formación:**
Pregunta: "Los operarios del cliente manipulan las celdas HV sin formación específica de seguridad."
- A) No es asunto nuestro → ❌ "La seguridad del cliente SÍ es asunto nuestro como Trusted Advisor"
- B) Safety Training → ✅ "¡Correcto! Formación en seguridad es una oferta clave de servicios"
- C) Cybersecurity → ❌ "Ciberseguridad protege sistemas digitales, no personas"

**Lead inválido 1:** "Renovación contrato que vence en 2 meses" → "INVÁLIDO: Renovaciones de contratos NO cuentan como F2O"
**Lead inválido 2:** "Lead cambio componentes sin confirmación cliente a +1 año" → "INVÁLIDO: Sin confirmación del cliente para ejecución a más de 1 año, el lead no es válido"

### Mundo Final — Reunión con Facility Manager:
Formato diálogo RPG. El facility manager hace comentarios/preguntas, el jugador elige respuestas.
Se evalúan 3 métricas: Confianza del cliente, Leads propuestos, Calidad técnica.
Perfiles finales: "Trusted Advisor" (ideal), "Vendedor agresivo", "Técnico invisible".

---

## REGLAS DE DESARROLLO

1. **TODO EN UN SOLO ARCHIVO index.html** — el script de Phaser se carga por CDN, el código del juego va inline
2. **Sin imágenes externas** — todos los sprites se generan por código en BootScene
3. **Usar Phaser Arcade Physics** para toda la física del plataformero
4. **Usar Phaser Scenes** para cada pantalla del juego
5. **Código organizado** — cada Scene como una clase separada dentro del mismo archivo
6. **Responsive** — usar Phaser Scale Manager con FIT + CENTER_BOTH
7. **Para ejecutar:** abrir con servidor local (python -m http.server 8080, Live Server VS Code, o similar)
8. **El idioma del juego es ESPAÑOL** — todos los textos visibles en castellano

---

## PLAN DE DESARROLLO ITERATIVO

### Iteración 1 (PRIMERA SESIÓN):
- BootScene genera todas las texturas por código
- TitleScene con pantalla de inicio
- BriefingScene para Mundo 1
- GameScene con Mundo 1 (Subestación LV) completo y jugable
- QuizScene funcional con las 4 preguntas del Mundo 1
- 2 leads inválidos como trampas
- Charcos de agua como peligro
- HUD completo (vida, leads, timer, bonus €)
- SummaryScene con resumen de puntuación

### Iteración 2:
- Mundos 2 (Data Centre) y 3 (HV) con sus quizzes y trampas
- EPIs como coleccionables
- Más variedad de peligros y entornos

### Iteración 3:
- BossScene (diálogo con Facility Manager)
- FinalScene con perfil del técnico
- Polish visual y balanceo

### Iteración 4:
- Testing, ajustes finales, versión offline para el día de la formación
