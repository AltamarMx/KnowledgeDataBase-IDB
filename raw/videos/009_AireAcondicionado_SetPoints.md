recording. Okay.

Bueno, primero, híjole, ya se acabó el semestre. Bueno, yo siento quedan 4ro semanas porque quedan cuatro semanas. Estaba revisando o se me quedan cuatro clases o cinco clases porque hay un puente, hay un viernes 15. Entonces, claro, para mí es, yo estoy pensando en semanas de clase, ¿no? Tal vez son cinco semanas porque es la última semana terminan las clases y empiezan de mayo y empiezan los exámenes. Yo en lo personal nunca doy clases, o sea, y tengo que aceptar si me retraso, ¿dónde me quedo? No, porque sé que no todo el mundo respeta y se vuelve llevar exámenes con clases. Complicadísimo.

Entonces, bueno, eso es eso es lo primero. Les voy a dar clase el miércoles. Este, ya les comentó Miriam, ¿no? Entonces va a ser una clase teórica y de Python para Ener Habábitat. Y revisando el temario, nos faltan, realmente nos faltan tres temas, ¿no? Eh, el de hoy va a ser uno, son schedules y aire acondicionado de simulaciones, ¿no? Eh, si bien el proyecto, no les vamos a pedir que hagan simulaciones con aire acondicionado porque eso otra vez duplica todos los casos. O sea, así de fácil, ¿no? Si si vamos a decir un caso base, tres estrategias más todos los casos, son cinco simulaciones por aire acondicionado, entonces son 10 simulaciones y lamentablemente, pero es lo que quiero plantear hoy, las mejores estrategias bioclimáticas para aire acondicionado no necesariamente son las mejores para sin aire acondicionado. Entonces, ahí hay que tener mucho cuidado porque justo justo ese ha sido como un error mucho tiempo de la norma de la NOM 008 y la 020. es que tenía que usarse la normativa, independientemente de si se usaba aire acondicionado o no. Ya después la reformaron y dice en climas cálidos y es aire acondicionado eh la energía para ahorrar eh energía de enfriamiento. Antes no decía y era es como un absurdo, uno de los muchos que tiene la norma. Eh, entonces, pues para no replicar, porque si no se les duplica, literal se les duplica el trabajo. Este, y para nosotros, como siempre tenemos el sueño de cambiar la vivienda social, pues nos dedicamos a sin aire acondicionado y además creemos que se puede, pero sí vale la pena tener claro cómo se hace, porque hay algo que son los schedules, que son los horarios, que vale mucho la pena. los schedules. O sea, ya hemos he platicado que Energy Plus me permite o nos permite implementar cargas térmicas, o sea, equipos, personas, apertura de ventanas, funcionamiento y eso se tiene que dar por medio de un horario. Entonces uno puede decir de las 8 de la mañana a las 12 sucede esto y y así, ¿no? Cuando uno crea un aire acondicionado, uno también puede decidir prender o apagarlo o hacer que funcione en los famosísimos set points. El set point es esa temperatura a la cual el aire acondicionado funciona, ¿no? Pero hoy vamos a ver las particularidades que tiene Energy Plus. Vamos a usar un sistema de aire acondicionado ideal. Eso quiere decir que la energía que proporciona es la energía que usa, es decir, la eficiencia es 100% o la fracción es uno. Eh, que si yo le pido 1000 de energía en las unidades que ustedes estén pensando, me va a dar 1000 de energía y si le pido 10,000 me va a dar 10,000. En la realidad eso no sucede así. Los aires acondicionados tienen una capacidad, ¿no? Y ese es uno de los trucos cuando uno diseña para aire acondicionado, uno permite, es como los sistemas fotovoltaicos, uno dimensiona de acuerdo al a la carga pico, ¿no? En potencia, entonces los aires acondicionados igual van a tener un pico de enfriamiento o una potencia máxima de enfriamiento y si la sobrepaso no voy a ser capaz de cumplir esos esa esta temperatura que yo deseo. Estamos platicando con el MUAC, ubica el MUAC, el Museo Universitario de Arte Contemporáneo de la UNAM. Ah, el MUAC tiene nueve salas. Está muy chido. Tiene nueve salas y las salas, Dime. Ah, sí, sí. y y todas las salas porque tiene pues obras de arte deben estar a 20ºC y no me acuerdo a qué porcentaje de humedad nos dijeron, ¿no? Eso quiere decir si estuviéramos en un clima muy difícil que a veces tendría que ser necesario calentar. De hecho, no se me ocurrió preguntarles, pero a lo mejor tienen calentamiento porque nos vamos a hacer un equipo del instituto y vamos a trabajar con ellos para ver si lo podemos hacer más sustentable el MOAC. Eh, pero no todos los casos son así. O sea, el MAC tiene condiciones muy específicas de temperatura y de humedad. Eso se puede lograr con equipos, ¿no? Pero eso quiere decir que si la temperatura baja, o sea, si fuera estuviera a 10ºC, pues entonces uno tiene que calentar. En la realidad y en México eso casi nunca sucede, o sea, tener un sistema que caliente y que enfríe, porque muchas veces las cargas térmicas compensan, pero no tenemos un control. en otros países, en Europa, en en Asia, en China, ¿no? Bueno, China es este Asia, ¿no? Pero eh o en ot en otros países sí hay sistemas de aire acondicionado que trabajan para enfriar o para calentar y entonces quiere decir que mantienen la temperatura en un punto fijo y eso es increíble desde el punto de vista de control. Pero imagínense el consumo energético y y también lo que estamos hablando es que a mí me gusta estar y voy a decir a 22ºC, no a 22.5 sino a 22. Pues eso digo, está muy chido, es primer mundo, pero lo cierto es que tenemos una zona de confort, ¿no? Los modelos adaptativos me dicen dependiendo de cómo yo esté acostumbrado, el lugar en donde viva, mis actividades físicas, mis hábitos de vestimenta, me van a decir que yo puedo estar entre 20 y entre 24, ¿no? Y a los 25 sí ya me empiezo a sentir incómodo. Entonces yo puedo mantener la temperatura en ese rango. Y entonces eso también es uno de los principios que usamos para diseñar que vamos a tener ese rango de confort. Ahora, cuando uno habla de rango de confort, eso quiere decir que no toda la gente va a estar confortable. Uno espera que al menos el 80% de la gente esté confortable a esa temperatura. Nunca vamos a lograr el 100%. Bueno, es muy difícil satisfacer a todo el mundo. Eh, entonces vamos a empezar simulando un aire acondicionado con una temperatura fija. Eso quiere decir que la va a mantener, pero entonces vamos a tener cargas de calentamiento y cargas de enfriamiento. Y después como segundo paso vamos a definir un set point, perdón, un rango de temperatura y entonces va a calentar cuando va quiere bajar y va a enfriar cuando quiera subir. Y luego vamos a hacer el tercer y último ejercicio. No quiero que haya aire acondicionado de calentamiento porque esa es la realidad de México. Entonces, ¿cómo tengo que engañar a Energy Plus usando el mismo sistema? Porque el aire acondicionado ideal solo me permite debo debo definir dos temperaturas, ¿no? Y tal vez ya se la están imaginando y la verdad es que es un trucazo, o sea, no hay nada este que tengamos que hackear, es nada más así como, ah, pues, ¿qué tengo que hacer? y y lo vamos a a y pues y si ya tenemos aire acondicionado, entonces también vamos a preparar nuestra salida para eh que tenga la energía de enfriamiento y la energía de calentamiento y como siempre cerciorarnos que todo está bien, ¿no? Siempre la ya lo dije la vez pasada, pero pues lo voy a volver a repetir un poquito, que la parte de visualización de datos es imprescindible para darme cuenta que estoy haciendo bien las cosas y más cuando y y dejen lo digo así, yo no tengo idea de cuánto consume un aire acondicionado en una casa porque nunca he usado aire acondicionado, ¿no? Eh eh tengo una idea de cuánto consumo en electricidad mi casa, no tengo aire acondicionado. Entonces, de repente, si yo les digo, "Ah, sí, claro, 300 megjols o mega kilw hora, pues probablemente si yo hago la integral no voy a tener la certeza de que está bien, pero si veo el comportamiento y sé que es lógico a lo que yo espero y que están los horarios, etcétera, entonces ya tengo una certeza." Claro, en algún momento sí tendremos ya cuando estemos diseñando tendremos que saber que esas energías que nos están saliendo son reales, ¿no? Eh, en la actualidad lo que la gente hace en ingeniería son reglas de dedo, es un espacio de tal, un aire acondicionado de tantas toneladas, ¿no? Pero si su envolvente está bien diseñada, puede ser que no necesiten tanto. O si su envolvente está muy mal diseñada, entonces lo contrario. Y y no solo se trata de ahorrar energía. Si yo disminuyo la capacidad de mi aire acondicionado, me voy a gastar menos. No es lo mismo comprar un aire acondicionado de media tonelada que uno de 3 toneladas o de 5 toneladas, ¿no? Este, yo no sé por qué usan toneladas. Yo creo que es una deformación gringa, este, de la capacidad de aire que puede enfriar en una hora, algo así, ¿no? Que puede el flujo de aire que puede mover y enfriar. Es algo así. Eh, entonces sí el diseño, el dimensionamiento de los sistemas de aire acondicionado es esencial, pero algo que voy a decir y que y que siempre vale la pena reflexionar es que, por ejemplo, hay normativas como lead que te dan puntitos y logras clasificaciones y la gente paga para tener desde desde dentro de esto que es el green wash. O sea, yo me parece que el ladit o algunas certificaciones pueden caer y sobre todo en México las certificaciones casi siempre nacen queriendo beneficiar algo, o sea, a la economía, ¿no? Que que gasten menos el consumo energético, pero lo cierto es que se pueden contaminar. Y les voy a contar un caso que sucede aquí en México y debería dejar de grabar para que esto no se eh, por ejemplo, sale la NOM 008 que me dice que debo usar este a aislante este y la gente la empezó a usar indiscriminadamente, sobre todo en edificaciones públicas porque es normativa se publicó en el Diario Oficial de la Federación y deberíamos todos los que construimos deberíamos pasar la norma normativa aquí no la tenemos, ¿no? Afortunadamente es el estado el que se encarga de revisar las normativas y entonces pues hay cosas que el estado no se da abasto y entonces no hay una comisión que que que revise que se cumplan y debería tener cada edificio nuevo debería tener un sellito, ¿no? De de que pasa la normativa. al principio de la normativa se empezó a usar para todos los edificios o que todos los edificios deberían usarlo y el primer error que se veía era que todo el mundo decía, "Es que vas a ahorrar energía." Y le decía, "Espérate, pero si mi edificio no tiene aire acondicionado, entonces el ahorro no existe. Bueno, pero vas a estar más confortable." Hm, pues podría ser, pero resulta que si yo pongo un aislante es posible, no voy a generalizar, que el flujo de calor que se genera aquí adentro no pueda salir porque tiene un aislant ahí afuera y y se va a almacenar aquí adentro. Entonces, esto se podría volver caluroso. Y lo que pasa es que el aislante funciona para ciertos climas, funciona para ciertas cargas térmicas, pero no siempre. Pero la industria, es decir, los que hacen pan el rey, los que hacen aislantes, lo ven como una oportunidad y empezaron a impulsar normativas, este, y y que la gente las cumpla y no solamente la gente, las empresas. ¿Por qué? Pues, ¿por qué van a vender más? porque así tienes que construir. Entonces, y y pregúntenles a los que impulsaban la industria del panel rey, si tienen noción de transferencia de calor, el modelo de dependiente contra el independiente del tiempo, no tiene idea. Y esto hace 10 años era también relativamente nuevo, ¿no? Entonces, si es, o sea, si no hay la información suficiente y ese es el papel de la academia y de la investigación, pues algún impulso de normativa con una buena intención se puede contaminar por los intereses económicos y entonces ahí andaban los de las normativas y lo siguen impulsando, ¿no? Porque es un negocio, ¿no? Y en muchos casos está bien, pero también hay muchos que está que vienen siendo contraproducentes. Entonces, otra vez, poder evaluar normativas o edificaciones y decir sí o no es indispensable. Y algo que también es indispensable es usar los modelos de transferencia de calor dependientes del tiempo, es decir, Energy Plus, ¿no? Bueno, entonces dicho esto, este, pues es super importante que sepan diseñar con aire acondicionado, ¿no? Y no solo que sepan, sino que, por ejemplo, hay un ejercicio y yo creo que ese sí se los voy a dejar de tarea, que no da el mismo resultado poner el aislante en cualquier posición. Si yo lo veo desde desde el punto de vista de los modelos de transferencia de calor independientes del tiempo, es una resistencia térmica. La resistencia térmica no importa dónde la ponga, es una resistencia térmica. me va a dar igual en transferencia de calor dependiente del tiempo. Sí importa la posición, ¿no? Pero pero eso, por ejemplo, ese es uno de los absurdos de la NOM, no te dice dónde. Afortunadamente casi todo el mundo la pone en el exterior porque es más fácil, pero nos hemos topado de casos que también es más fácil ponerla en el interior porque te quitas el mantenimiento. Si pones en el exterior te va a degradar por el clima, por quien se suba a la azotea o algo. Cambio, si la pones aquí adentro pues no. Pero podría haber un gran cambio, ¿no? Entonces, esa va a ser una tarea. Okay. Entonces, eh voy a voy a agarrar el el clima. Se ve bien. Sí, va. Sí. Voy a agarrar el la simulación. Déjenme cerrar todo esto,

la simulación de que hicimos la vez pasada y voy a agarrar el de dos ventanas y voy a crear una simulación con aire acondicionado. Sí. Entonces, a ver, ahí voy.

Entonces, CD desktop, dos zonas.

Aquí está mi OSM. Creo que me sí me vale la pena hacerlo un poquito más grande. Si hasta yo lo veo chiquito, nada más. Appearance.

Me acabo de cambiar a la MAG y ya no encuentro dónde se pone el tamaño. Hay una parte donde puedo poner el tamaño.

A ver, seis.

Déjenme googlearlo. Increase

size sintajoe

system accesibility

y display.

A ver, display to adjust displ resolution for do size in settings. size

No manches.

System setting display scalet resolution scale.

Aquí está. Ah, pues lo tengo que cambiar la resolución por eso yo creo que 1600 por 1900. Sí se ve mejor o le subo todavía. Sí. Okay, déjenme ver algo si OBS no sé. Okay, parece que no se rompió, que sigue grabando toda la pantalla. Muy bien, entonces vamos a voy a agarrar el caso base, ¿no? Sí, ya no voy a sufrir. Este la la M5 Pro del instituto. Eh, la M5. Okay. Eh, voy a guardar este. Acuérdense que lo que debo hacer es guardar como y entonces ya tengo un seis. Entonces voy a hacer un 07 caso base gu bajo aa de aire acondicionado. Me voy a acerciorar que funcione

y

ahí está. No, parece que sí funciona. Le doy show simulation y está el err que lo debo abrir con el text edit pqrs text edit. Y ahí está. Entonces, pues se ve se ve bien, ¿no? Eh, warning, gurus face output. Y fíjense como justo hice un cambio de computadora, entonces todo funcionó. O sea, ¿por qué? porque agarré el folder y las rutas siguen funcionando. Entonces, esa es una otra vez esta de las cosas de por qué es importante, es un espacio de trabajo porque me permite moverme si las cosas están bien hechas, sin mucho esfuerzo. Okay, entonces funciona. Lo primero que tengo que hacer

es activar el aire acondicionado. Entonces, yo tengo dos zonas térmicas, ¿no? Y si se fijan aquí en la pestaña esta de Thermal Zones tengo un prender aire acondicionado ideal, ¿no? S es ese voy a voy a prender los dos, voy a crear dos este diferentes nada más como para hacerlo. Entonces lo prendo y en teoría ya tengo un aire acondicionado ideal. Entonces, la verdad es que es ser sencillo. Si es mucho más complicado hacer un aire acondicionado sin real, perdón, real. ¿Por qué? Pues porque tengo que poner ductos, tengo que poner un ventilador, tengo que poner un aire acondicionado. Luego hay un tema de recirculación. Por ejemplo, los aires acondicionados, los mini splits solamente recirculan aire, toman aire de aquí y recirculan. Pero los, no sé si cuando la pandemia se volvió super evidente que iban a decir los cines que iban a permitir entrada, aumentar la para aumentar la renovación porque recirculan una fracción y sacan otra o toman una fracción del exterior y entonces siempre están calentando una pequeña fracción del exterior. Este aire acondicionado ideal lo que va a hacer es va a calentar solamente el aire del interior. Es como si fuera un mini split, ¿no? Entonces, hay que estar como como conscientes, ¿no? Eh, entonces ahí ya tengo y lo que y pero por eso ven que tiene air loops on equipment, o sea, y aquí yo debo confesar eh, yo no sé simular aires acondicionados reales y y es muy claro porque pues mis intereses no son industriales, ¿no? Vamos a ver. Este sistemas de climatización geotérmicos como los de la oficina se pueden simular aquí los sistemas de ¿qué? Ah, los como cuáles como los como de la oficina de de Sí. O sea, que que tienen el subterráneo. Sí. Para que la circulación. Bueno, la entrada, o sea, Energy Plus. Primero, por ejemplo, si quisiéramos simular un algo real, primero tenemos que meter la ventilación, ¿no? Y eso se mete con airflow network, no lo vamos a ver aquí, lo vemos en la siguiente materia. Este, y si yo quiero conectar un objeto de aire acondicionado, tengo que verificar que sea compatible con Airfrost Network. Y no todos son compatibles porque hay lugares que están sellados literal y que no tienen ventilación y eso sucede más en Estados Unidos. Otra vez, aquí en México no estamos tan acostumbrados a eso. A mí me ha tocado ver, no sé, el archivo de no sé qué del Instituto Federal de Telecomunicaciones y estaba sellado, no había ventilación. Entonces eso ciertos tipos de aire acondicionado que no permiten lo hacen y hay pero sí hay aeros acondicionados que te permiten interaccionan con la ventilación y y hay uno de tubos enterrados, eh, y puedes tú definir, si recuerdo bien, que son verticales o horizontales, porque es diferente y entonces tú dices, ¿cuál es el flujo de aire? Y entonces puedes hacer el cálculo y o sea le dice tipo de aire, pero ¿cómo toma los datos de la temperatura del sexual? Del EPW. Ah, ese es todo un tema. Eh, del EPW puedes hacer una suposición, pero resulta que la temperatura del suelo va a depender del material que esté hecho, o sea, si es roca, si es lodo, si si hay humedad. Entonces, lo mejor que se puede hacer es medir. O sea, hay gente que está tratando de adivinar, pero pero si te cambia el suelo, hay hay una regla que tú pones la temperatura como la temperatura, pero pero esa es la temperatura del suelo de condición de frontera y que es diferente a la temperatura a diferentes profundidades porque conforme te vayas adentrando la temperatura va a oscilar menos. Entonces, Energy Plus sí tiene modelos, pero lo mejor es que tú le digas la temperatura. Ah, sí, aquí ya sabemos que a 1.8 m está integrado con Sí. Dile a Hón que sí se puede hacer eso, ¿eh? y y lo y se puede hacer, ¿no? De hecho, por ahí hemos cuando hicimos el edificio este se hicieron unas primeras simulaciones, pero otra vez es demasiado y y en ese momento pues eso fue hace como 5 años, ¿no? no había, no tenía tantos datos y y además cavar pozos no es barato, o sea, es muy barato y piénsenlo, cabar en vertical, pero si yo voy a poner una red en horizontal, en el plano horizontal, quiere decir que tengo que levantar todo ese espacio, ¿no? Entonces realmente llegar con un taladro y cabar en en vertical es lo más sencillo y barato. Y aún así no es barato, ¿no? Porque si te si te bueno en seu que tienes roca volcánica se vuelve complicadísimo, ¿no? Este o técnico, no sé qué tanto, pero bueno. Entonces esas cosas se pueden. Eh, hay lo mejor es medir y pero también el comportamiento y la difusividad y todo eso. Entonces, muchas veces también vale la pena hacer pruebas de propiedades de propiedades térmicas, ¿no? Conductividad, densidad y calor específico. Los materiales ahí. Okay. Entonces, pues ya activamos. Si se fijan, tenemos que definir dos cosas, un schedule de enfriamiento y un schedule de calentamiento. Y entonces ahí era lo que lo que les decía. Déjenme jugar con mi juguetito nuevo y le creo que tengo

ahora sí vengo. Listo, creo.

Y ay, ¿cómo me voy al al

whiteboard? Okay. Entonces, por ejemplo, si

si yo tengo, déjenme rayar y voy a agarrar

si yo tengo. Yo creo que sí está bien.

Ojo.

¿Qué? Okay, ya. Ah, pero tengo que hacer que no se desvanezca.

Okay. Si yo tengo una temperatura de set point, ¿no? Y si mi temperatura está oscilando así. Entonces, pues realmente es es relativamente fácil pensar que, déjenme lo hago más chiquito, que por ejemplo aquí tengo que llevar la temperatura de aquí hacia acá, ¿no? O sea, tengo que calentar y aquí tengo que enfriar, ¿no? De tal manera que tengo una un punto y entonces yo podría definir un set point a una temperatura. Vamos a suponer que este está a 20ºC. Entonces, si yo defino y no voy a poner calentamiento y enfriamiento porque podría ser confuso, porque luego es, okay, ¿y a a qué temperatura están esos set points? Entonces, los voy a definir a temperaturas. Pero pero fíjense, si yo pongo un mismo set point para enframiento y para calentamiento, es lo que va a hacer esto. Si yo en algún momento, y creo que lo que sí no puedo hacer es borrar, déjenme ver cómo cómo

clear screen. Okay. Otra vez si yo tengo

No, así no va. Control Z. Sí, si va así mi temperatura, pero en lugar de de definir una temperatura, pues ahora hablo de un un una zona de confort, entonces yo defino y puedo tener diferentes, digo. Entonces, vamos a suponer como usé uno a 20, este es 20 gr cent y este de aquí abajo vamos a ponerle 15.

Sí, 15º CC. Entonces, otra vez va a pasar lo mismo. No voy a poner temperaturas, voy a poner valores, pero sé que esto me va a servir, ¿no? O sea, cuando la temperatura baje y cuando esté en esta zona, no va no va a tener no va a permitirle oscilar. Entonces, ¿qué voy a ver? Pues yo voy a ver que la temperatura oscila y luego se mantiene aquí constante y luego va oscila hacia abajo si es que es necesario, ¿no? Cuando tengo una zona. Y entonces ahora ahí les va el último tip, que es el último caso que vamos a hacer. Este y realmente todo esto va a ser super rápido, ¿no? ¿Qué tengo que monitorear? La temperatura de las zonas térmicas. Si yo no quiero que haya calentamiento o que no haya enfriamiento, ¿qué tengo que hacer? ¿A quién se le ocurre? Si yo tengo, si yo quiero que no haya calentamiento o que no haya enfriamiento, ¿qué tengo que hacer? Los tengo que definir. Esas dos temperaturas tienen que existir, ¿no? No las puedo dejar. Si las dejo en cero, se me va a bajar a cero, ¿no? Si las si las dejo indefinidas, como está ahorita en la simulación, me va a marcar error. Eh, si las pongo iguales, me va a ser me va a hacer una línea recta porque porque lo va a mantener a la a esa temperatura, ¿no? Porque no lo no va a dejar que se enfríe y va a calentar. Ajá. Entonces, ¿qué hago?

Es como problema de lógica.

No quiero que caliente o no quiero que enfríe. Vamos a suponer que no quiero que caliente porque en México casi nunca hay sistemas de calefacción. Entonces, ¿qué hago con el 15 gr? Porque lo tengo que poner. Tengo que poner mi set point de cooling, mi set point de de heating. O sea, si está abajo de 15 va a calentar. Si está abajo de 15 va a calentar. Ah, pues pones los límites muy pones el límite de muy muy inferior, muy bajo. Ajá. O sea, si le pongo que que no que caliente cuando llegue a 0 gr cent y nunca va a calentar si estoy en Temisco, porque si estoy en Nueva York sí va a calentar, ¿no? Y y lo mismo no quiero que haya enfriamiento porque estoy en Toluca, entonces pongo el sistema de de enfriamiento a 50ºC.

Ajá. Entonces, es lo que les decía, o sea, siempre hay maneras de hackear porque, ¿qué tal si le dicen, "Ay, no, no quiero que haya tú?" Ah, en lugar de hacer una simulación donde quitas el sistema de aire acondicionado porque quieren los resultados ya, pum, le pones un set point que nunca funcione, después sabes que tienes que quitar ese sistema de aire acondicionado, ¿no? Lo que haga entonces realmente los tres ejercicios, pero lo que quiero que vean hoy, además, o sea, además de de del de la lógica atrás del enfriamiento y calentamiento, es cómo se manejan las schedules, ¿no? O sea, ahí lo que me dices es que necesito un set point y para crear un set point necesito un schedule. Y lo fantástico de Energy Plus es que yo podría definir un schedule que cambie cada minuto, ¿no? Yo podría diseñar un sistema eh así como pixeleado que la temperatura siguiera una onda sinusoidal si es que yo tengo la paciencia de definirla así, ¿no? o decirle justo, o sea, no quiero que haya aire acondicionado eh de 8 a antes de las 8 y después de las 6 de la tarde, antes de las 8 de la mañana, que es el horario de clases, suponiendo que aquí hubiera aire acondicionado. Entonces, pues tengo que hacer un schedule que se que los límites afuera de eso estén super arriba o super abajo para que no pase nada, ¿no?

Okay, creo que esto voy a terminar acá.

Y entonces, fíjense cómo me piden un cooling thermostat y un heating thermostat. Hay también sistemas humidificadores, pero esos no los vamos a tocar ahorita. Sí, acá arriba en la segunda pestaña vean como hay que dice schedules. Y lo fantástico de Estados Unidos y de Asri es que ya tienen tipos de schedules y de cargas térmicas para un montón de casos, de escuelas, de oficinas chicas, medianas, grandes, industrias, es este hospitales, porque tienen cargas cargas energéticas muy diferentes. Justo algo que estamos tratando de hacer con la tesis de Eric Iván es, ¿cuál es el consumo eléctrico de las casas? Porque el consumo eléctrico de las casas va a definir de alguna manera su comportamiento térmico, porque el consumo eléctrico es luz, es tele, es lavadora, es refri, que son es es bueno, cocina, si fuera una una casa con con cocina eléctrica. Y todo eso se transforma en cargas térmicas y las cargas térmicas transforman el comportamiento de la casa, ¿no? Eh, lo malo es que ustedes van a simular una casita sin cargas térmicas, ¿no? Okay. Entonces, le voy a decir que quiero un nuevo schedule. Voy a crear, había dicho 20 y 15. Déjenme subirlo. Lo voy a poner 24 y 20. No, de hecho a mí me da frío como abajo de 18ºC, ¿no? Alrededor de 18, pero vamos a ponerlo 20 20 25 para que sean numeritos cerrados y los podamos visualizar muy fácil. 20 25 20 de calentamiento, 25 de enfriamiento. Entonces, primero voy a crear un set point a 20ºC.

Ah, perdón, estoy en la pestaña de schedule sets. Entonces, como ya conocen el concepto de sets, que es que puedo hacer un conjunto de schedules para diferentes tipos de espacios, pues esto es más o menos lo mismo. Entonces, me tengo que ir a schedules, le doy en el más y fíjense, lo primero que me haces es me pregunta, ¿qué tipo quieres? Yo puedo decir prendido y apagado o dar fracciones cuando es la ocupación. Nada más se los voy a poner. Por ejemplo, yo puedo definir una ocupación de cuatro y si quiero que entre una persona, lo multiplico por punto 25 y entonces tengo un uno. ¿Qué es lo malo y lo peligroso? Pues que podría yo, si hago mal mis cuentas, tener 1.1 personas, ¿no? Y Energy Plus no se va a quejar. Lo único es que va a meter la carga térmica de por a través del metabolismo de 1.1 personas, ¿no? Eh, entonces hay de dimensionless y hay uno hasta abajo de temperature. Hay diferente de temperatura y y aquí no se ve, pero es que hay unos que tienen límites y otros que no tienen límites. Entonces agarramos el primero de temperatura y y fíjense como lo que me dice es numeric continuous lower limit non uper limit non y lo creamos y ya está que está s super poderoso porque yo puedo vean cómo tengo un room period file profiles yo puedo definir ese es mi lo se le llama el default ¿no? Y después los schedules se van encimando. Entonces lo si yo nada más defino uno, va a quedar ahí, pero yo puedo definir para periodos vacacionales, etcétera, ¿no? O sea, con todo el detalle y lo puedo hacer prácticamente día por día, ¿no? Entonces, eh le voy a poner aquí el principal porque nada más voy a hacer uno. Enter. Y y aquí viene lo complicado de esta interfaz. Yo espero que haya un representante de cada equipo, al menos, ¿no? Porque esta es la parte tricky. Ah, bueno, no van a ser aire acondicionado. Estea

porque por porque si no se van a duplicar. O sea, habíamos dicho eso, eso lo platiqué hace ratito. Si yo hago, o sea, ustedes van a hacer caso base, estrategia 1, 2, 3 y c todas sin aire condicionado. Y si te pido con aire condicionado, entonces ya tienes 10. O sea, con aire condicionado más estrategia 1 2 3. Y además lo que comentaba al inicio de salud de la clase es que luego la mejor estrategia para aire acondicionado no son no es la mejor estrategia para aire sin aire acondicionado. Entonces vas a tener que pensar esas estrategias. Yo si quieres hablo con Miriam y les dejo les dejamos eso, pero es demasiado. O sea, el objetivo de esta materia, o sea, para nosotros queda muy claro es que aprenda enseño bioclimático y cómo evaluar. Pues va a ser imposible que puedan evaluar todas las estrategias, pero pero la y y los cambios, ¿cuáles son? Bueno, además de que son el doble de simulaciones, en una evalúas el disconfort térmico y en la otra evalúas consumo energético. Y siendo honestos, evaluar consumo energético es bien fácil, o sea, es agarras la columna punto zoom, ya está, ¿no? Entonces también es como, pues, o sea, lo vamos a hacer, justo eso lo vamos a hacer hoy, ¿no? Pero no se los vamos a pedir en el en el proyecto final, ¿no? Porque si no no les da tiempo. Bueno, sí les da tiempo, pero para qué los hacemos sufrir tanto, ¿no? Okay. Truco de esta interfaz, ese valorcito que dicen ahí, lower limit y limit. Eso no tiene nada que ver con el con el valor del del aire del set point. Acuérdense que vamos a definir un set point. Esa línea negra es el valor del set point. Entonces, ¿qué tengo que hacer para cambiarla? Simplemente pongo el mouse sobre ella. Ajá. Si pones esto se mueve, pero pero si te fijas no se está moviendo el valor. Mira, si yo veo acá me dice que es 23.5. El valor no se mueve. Cuando yo me pongo en medio, encima, ahí escribo el valor. Ajá. Pero ahí escribo el valor. Entonces, por ejemplo, habíamos dicho que era 25. Entonces, ahí va 25. Fíjense cómo está escrito ahí. y enter. Ahora sí está en 25 y ya. Ahora les voy a contar algo que podemos hacer, pero que no lo vamos a hacer, al menos no en el proyecto final. Bueno, ya les dije que no vamos a usar aire acondicionado. Si yo quiero decir, "Ah, pero a las 8 de la mañana no quiero que haya, entonces me paro a las 8 de la mañana, está aquí, y le doy doble clic." Y vean. Entonces, ya tengo dos segmentos y este segmento le digo, "Ah, pues vete a 80." Vean. Entonces, si ese yo lo dejara, va a generar 80ºC al interior y luego va a bajar a las 8 de la mañana 25, ¿no? Obvio no lo quiero así, lo quiero deshacer. Entonces, le voy a decir que este se vaya a 25, enter. Y le doy otra vez doble clic acá y desaparece el corte. Aquí abajo es como hacer un zoom. Aquí me permite ir cada hora, cada 15 minutos o cada minuto. Acuérdense que Energy Plus me permite hasta 60 steps por hora, es decir, un minuto. Entonces, no puedo hacer una resolución más pequeña de un minuto. Cualquier cosa que pase en menos de un minuto energy Plus, no lo pueden simular, ¿no? Porque solamente pueden simular un minuto. ¿Qué tendrían que hacer? Por ejemplo, si cocinas 10 segundos, puedes asumir que este tiempo que cocinaste lo tienes que repartir en un minuto, ¿no? Okay. Entonces, este es mi principal 25ºC. Tengo que crear otro schedule. Voy a crear uno nuevo. Se acuerda que es de temperatura. Y entonces simplemente me va me paro aquí, le pongo 20 y le pongo acá 20 gr cent. Pum. Ya tengo dos schedles.

Ah, le puse este, perdón, le puse principal. Entonces, este es el de 25 y este es el de 20. Le puse mal nombre. Sí, ahí está. Lo reviso. 20. Ahí está. 25. Ahí está. Eh, lo voy a guardar. Save as. No, no le voy a No lo voy a versionar. Me voy a venir a mis Thermal Sounds, que debe ser esta, y me vengo a Myodel. Y vean, ahí están los schedules. Entonces, voy a hacer eh pues la demostración de que si hago este en cooling y este cooling y todo a 20 grar.

Sí funciona.

Si lo malo es que mi compu ya no los va a esperar ahora, ¿no? Okay, ya terminó. No he sacado los resultados entonces y la verdad es que no me acuerdo. Entonces, vamos a ver el RDD. El RDD.

Choose application.

Text edit.

Ahí está.

Y lo que voy a buscar es uno que me diga cooling.

Fíjense cómo está. S a sensible cooling. Vamos a ver qué más hay. sensible cooling rate son wats son predicted sensible load to cooling set point heat transfer rat son pred sensible load son thermostat cooling point yo podría pedirle el set point por si quiero hacer cálculos no eh son ideal air sensible cooling latent total cool cool cooling aquí como que este Me gusta, ¿no? Eh, se lo voy a pedir en potencia.

Cooling sound ideal latent sound ideal total cooling. No, energy.

Mm, no creo creo si me voy a quedar. Creo que estos nada más nada más están en

Ahora vean cómo tengo una entre Zone Ideal Loads supply air y tengo un Zone Ideal load son total cooling energy. Puede ser que no sea lo mismo, porque el supply me da una idea de que es la energía que va a proveer el aire acondicionado, ¿no? Pero si hay otros equipos podría darme el total, ¿no? O dividírmelo. Entonces, ¿qué tengo que hacer? Pues irme a la documentación. Estoy copiando este, el Z idea load total cooling energy para ver si

si cómo se llama, buscarlo en el engineering reference documentation, en el input output

y vamos a buscarlo.

Aquí está Z, ideal load, Zone, total cooling energy. Y vamos a ver qué dice la documentación.

Is the total sensible más latente. M, suena bien porque si hay un cambio de fase ahí adentro, lo estoy considerando. Cooling energy del to. If there is no out the zone ideal load supplier total cooling energies igual pues parece que este es el que quiero. Sí. Entonces lo voy a ya lo tengo en el portapapeles. ¿Qué voy a hacer? Pues me voy a venir y voy a agregar un output reporting. Ah, seguro esto no tiene no tengo los measures.

Voy a agregar los measures de addiable

y realmente es la única que necesito.

Estuvo viendo, bueno, estuvo viendo la recomendaría. Yo he visto que está chida. Pero yo te diría que voltearas a ver las limitaciones de software, porque hay mucho software que en la Mac no hay y que usas en Windows y luego pasa mucho en el instituto. Por ejemplo, eh, déjenme reiniciar porque creo que me hace falta algo. La verdad es que para hacer simulaciones de Energy Plus, yo prefiero Windows. En esta clase no se va a notar porque solamente uso Open Studio, pero para la siguiente semestre necesitas el IDF editor y el IDF editor no corre en Mac, no existe, entonces pierdes. Entonces yo lo que hago normalmente es traigo una máquina virtual adentro de la MAC y eso está padre si tienes mucha RAM y mucho espacio y la Mac Neo viene limitadona. Entonces, pero pero eso te da una idea, o sea, esa es mi mi necesidad. Si en otra materia que te falta necesitas alguna libro, alguna paquete que esté en Windows, yo creo que a la Magneo le va a costar trabajo correr una virtual, una máquina virtual con parallel y la y además la vas a tener que comprar el Windows Parallel, ¿no? Bueno, esa sería mi recomendación, ¿no? Eh, entonces yo a esta eventualmente le voy a poner Windows, ¿no? O sea, que que corra Windows este para poder hacer eso, pero esta tiene mucha RAM, ¿no? Afortunadamente, ¿no? Entonces, sí, esa sería mi recomendación. Okay. de energía. ¿Cuál? Sí, las de energía edificaciones, ¿no? Sí, junto con Miriam también, pero Miriam nada más da como dos o tres semanas porque ella es la experta en Airflow Network y la verdad es que yo lo sé hacer, pero pues prefiero que sea Miriam la que la que da eso. Sí, déjenme ver si ya tengo el QAQC. Aquí está.

Lo voy a agregar.

Voy a voy a quitar los demás porque la verdad es que no los necesito para que sea una simulación limpia. Time step. Y ya está.

Ah, ya terminó. Okay, pues vamos a ver los resultados.

Eh, estoy aquí. V Run Jupiter Notebook. Fíjense, en esta máquina, por ejemplo, no tengo el ambiente virtual ahorita.

O sea, ahorita está está recreando el ambiente virtual, ¿no?

Ah, sí se tardó.

¿Cómo andamos de tiempo? Okay.

Ahí va.

Perfect.

Voy a crear una nueva libreta. La voy a crear desde cero 003 revisión un set point,

¿no? Entonces import

pandas pd fromat plotli punto p plot no

import

as plt fromols read Port read

SQL.

Okay. ¿Qué tengo que hacer? irme a, déjenme

el autoclose brackets. Tengo que salirme

y entrar a OSM y entrar al 007, al run E plus out

SQL y luego le voy a poner aa de aire acondicionado

RE SQL F alias

true punto data.

Ahí está.

De hecho, ah le quité la temperatura, me equivoqué, ¿no? Por eso. Entonces, pero ahí está. Entonces, pues me regreso a mi simulación. Eh, voy a agregar,

dejen lo pongo acá, library.

Ahí le podía poner sound min air air temperature y lo voy a copiar. Ya me lo sé este time y hago la simulación.

Okay, vuelvo a cargar y ahí está. Y vean, de hecho ya se ve está 20ºC, ¿no? Check, chequen. O sea, en serio que de repente uno se equivoca un montón en estas cosas. Entonces, eh, fic ax pl t suplots fix size

12,3

supplots.

Togles and mode y entonces ax plot a aati este label.

Ah, este

o este

o este. Pum. Cómo me aseguro que está bien, porque las dos están encimadas. Pues quito una. No.

Eh, de hecho aquí tengo un problemilla. Vean como uno por e cuando Matl tiene una línea y no varía, como que se equivoca en el zoom, ¿no? Eh, pero vamos a ayudarle. Ax. Yim

de 15 a 25. Ahí está. Entonces está en 20, ¿no? Yo podría decir a eh grid.

Ahí está 20. Y si quiero asegurarme que las dos están ahí,

esa la voy a hacer roja con punto. Y esta la voy a hacer negra con bolita con líneas punteadas.

Ahí está. Ahí están mis dos valores. No, no tengo ninguna discontinuidad. Entonces realmente funcionó como lo esperaba. Eh, tengo dos set points. Entonces, ¿cuál es la variante que me hace falta? Pues primero zona de confort, ¿no? Este lo puse a los dos al 20. El de 20 de enfriamiento lo voy a dejar, perdón, el de 20 de calentamiento lo voy a dejar ahí y el voy a poner el de enfriamiento a 25. ¿Sí? Entonces, solamente va a enfriar cuando la temperatura suba de 25ºC, ¿no? ¿Qué me tengo que hacer? Pues me vengo a mi simulación, me voy a las zonas térmicas y my model y entonces simplemente es lo quiero para que sea el de cooling. De hecho, ¿qué tal si lo hago solo en uno? No, ahí está. La zona este debería tener una zona de confort y la zona oeste temperatura constante. Hago la simulación.

Me vengo para acá, vuelvo a cargar mis datos

y vean. Entonces, ahora le voy a poner 30ºC. Ahí está. Sí, caso tres, pero tod bueno no sé si terminar la clase, no terminar la clase, estuvo muy rápida, pero eh

pues el caso tres es como la verdad es que a estas alturas es trivial, fíjense de todas maneras ya no me acuerdo qué clima tengo, pero déjenme ver si puedo jugar tantito. No,

que para eso,

o sea, lo que yo veo todavía no

se te vería, ¿no? Ah, sí, ahí está. O sea, este clima solo solo aquí es cuando está bajando de de 20 gr cent. Aquí si parece que no es por por el tamaño de la bolita, ¿no? O sea, realmente no hay Puedo hacer suma así, ¿no? No puedo hacer suma así. Eh, acá sí, ¿no? O sea, todo esto se ve muy claro, el el aire acondicionado. Entonces, aquí bastaría con que la ponga a 18ºC y no va a entrar, pero un poco es así como, ay, vamos a asegurarnos que no va a fallar. La verdad es que 0 gr cent es un clima difícil y es temperatura al interior, ¿no? O sea, que una casa alcance una pero seguramente en el norte sí hay. Eh, entonces siempre revisen su simulación. Y yo nunca he puesto números negativos, igual podemos poner ahorita -1. Entonces, vamos, vamos. Pero, ¿qué tengo que hacer? Pues tengo que déjenme

desactivar esto. Pues crear un schedule. Vamos a crear uno de menos un schedule. Voy a agarrar. Le digo que es de temperatura.

Apply. Me paro sobre esta y menos. Ah, no me deja uno. Ah, sí, sí, sí me dejó menos1.

Sí, no se ve la el el el la diagonal, pero sí se ve, pero pero sí lo hace. Y entonces este es un -1

y ya tengo un schedule.

Ahora

pues me vengo a la zona térmica.

Vamos a dejarle a la a la oeste

que el heating sea menos un, es decir, nunca caliente, ¿no? Y se debería ver un poquito diferente.

borro

y vean, de hecho funcionó superb. Sí, me hace falta un pasito de todo esto. Fíjense, fíjense. Por ejemplo, la zona Ax Label Ax Legend AX Legend,

o sea, la zona oeste la estoy enfriando a 20ºC,

pero la dejo la dejo que que baje la temperatura. La zona este la enfrío a 25ºC y no la dejo que que baje la temperatura, ¿no? Por supuesto, no puedo hacer comparaciones de una zona con otra porque tienen diferentes tamaños, orientaciones, etcétera, ¿no? Pero supongan que es la misma simulación, pues una va a consumir más energía que otra, ¿no? Una va a consumir energía de enfriamiento y calentamiento y la otra solamente energía de calentamiento. Pero justo esa sería una estrategia. O sea, la pregunta es que podrían ustedes responder en un futuro, ¿cuánta energía me ahorro si en lugar de poner el aire acondicionado a 21, lo pongo a 22ºC o a 23? O mejoran aún, ¿cuál es mi límite superior de confort? Entonces, lo puedo calcular con la temperatura con el modelo de home phrase, el delta de Morillón y lo pongo ahí y eso y a lo mejor lo puedo bajar, pero pero ese sería mi mi meta y lo puedo comparar contra la temperatura, los aéreos acondicionados, no sé si ustedes les ha pasado, pero luego es ridículo que que entras en los congresos pasa un montón y siempre nos quejamos. Guadalupe siempre se queja y es hay muchas cosas que admiro de Guadalupe, ¿eh? que dice, "¿Cómo es posible que tengamos que ir a la costa y para entrar al Congreso te tengas que poner una chamarra?" O sea, es ridículo, ¿no? Cuántas veces no nos pasa que a veces vas al cine y te tienes que llevar una chamarra porque sabes que te va a dar frío, ¿no? Y eso que es en la Bueno, a veces no, yo siempre casi siempre voy en la noche, ¿no? Este, pero entonces vean, ahí tienen un gap de consumo de energía. que podrían estarse ahorrando. Claro, también pensemos en algunas cosas, ¿no? Y sería como democratizar el uso de la energía. ¿Para quién está pensado ese aire acondicionado? Porque hay gente trabajando ahí, ¿no? Y luego a la gente, por ejemplo, es una ridiculez, a mí me parece, porque en los bancos les obligan a ir de manga larga y saco a muchas personas, ¿no? Sobre todo las que tienen trabajo con clientes y ciertos clientes además. Y entonces eso hace que el aire acondicionado esté más para que estas personas no sufran. Y entonces tú entras media hora y media hora a lo mejor la aguantas. Pero si te quedaras así como andamos nosotros del diario, nos daría frío. Entonces, bueno, ahí la respuesta no es fácil, pero tendría que los cambios desde dónde tienen que venir. Okay, pueden ir a trabajar y puedes llevar camisa de manga corta y está bien. Okay. No te voy a permitir que lleves chanclas y y y bermudas, ¿no? Porque quieres un ambiente formal, pero sí, camisas. Eso lo hizo Japón. Japón por medio del emperador hizo un decreto que la gente podía dejar de llevar o que debería llevar dejar de llevar de dejar de usar saco y corbata para poder reducir el set point del aire acondicionado y ahorraron energía, ¿no? Eh, claro, ahí son las ventajas de tener un emperador, ¿no? Este, en algunos casos. Okay. ¿Qué me hace falta entonces? Pues ah, tal vez a esta gráfica lo único que voy a hacer, voy a agarrar una temperatura, la voy a copiar, me voy a quedar con una temperatura, la del la roja,

eh, lo voy a hacer un periodo de tiempo.

from date útil.

No, ya se me olvidó.

Sí, voy a voy a este importar el pars de date utils.

Entonces voy a poner mi fecha uno

del 2006

cero. Estamos que en abril 04 17 y luego mi fecha dos es igual a fecha 1 + PD delta. Voy a poner 10 días y a X set X lim de fecha uno a fecha dos. Ahí está. Voy a hacer un supplot porque quiero ver aquí la energía. Eh, cuando hago un suplot, fíjense, como ya tengo todas mis Ax definidas, yo puedo, si lo dejo así, en Ax recibe ax de corchete cuadrado 0, ax de corchete cuadrado de 1. Sí, ahí recibe los dos objetos, pero yo le puedo decir a como ya no quiero modificar a ax y ax2 y entonces esto sigue funcionando, pero ahora tengo un ax2. Entonces, en AX2 punto plot, esto no es lo mejor, o sea, desde el punto de vista de de de técnica, debería haber una X1 o una X2, pero a veces también hay que estar conscientes que uno lo que quiere resolver, sobre todo en una junta y quiere entonces esto para no estar así poniéndole el uno a tos mis, que también sabemos que lo podemos hacer, pero pero existen estas cosas y es a y voy a usar corchetes cuadrados porque quiero una variable que es el total cooling.

Mm.

Ah, ya me había espantado.

Ya vieron. Está padrísimo, ¿no? Vamos a ponerle 3 días.

Otra vez, ¿qué estoy haciendo? Revisando que mi simulación está bien hecha. Tengo mi temperatura que empieza a oscilar y entonces ahí no tengo consumo de energía, ¿no? Y después se mantiene constante y está el consumo de energía. Ahí se ve el consumo de energía máximo. Sí, si esto era energía, pues sí, ahí está en Jules, ¿no? Entonces, ya está. O sea, mi simulación se ve bien. Entonces, si mi simulación está bien, entonces ya me puedo atrever a hacer un cálculo, ¿no? No voy a convertir a renombrar esa esa variable esta, pero yo puedo hacer aire acondicionado, corchetes cuadrados. Es la del este.

¿Cómo hago? ¿Quién quién quién se acuerda cómo hago? Quiero quiero reportar. ¿Quién quiere pasar a escribir ahí tres líneas de código? ¿Quién quiere pasar a escribir tres líneas de código? El consumo mensual de la energía. Es interesante porque yo esperaría que en abril o en mayo sean los consumos mayores porque tengo energía de enfriamiento y entonces eso también otra vez me da una ¿Quién quiere pasar ahí? Nadie.

Son son una línea. Es una línea. Bueno, díctame. ¿Qué tengo que hacer?

Quiero el consumo mensual de energía. Punto. Eres ingeniero casi. Y ahorita me voy a ir así, mira. Si no, de todas maneras

horas con una línea de código. Fernanda,

yo esto lo hago todo el día, ¿no? Entonces, todos los días.

Es que fíjense un poquito qué pasa. De repente es bien fácil cuando yo se los digo, entonces si se los digo no piensan, no se les queda, ¿no? Entonces tienes las horas. Tengo las horas siempre. Bueno, mis lo que yo tengo de ear tools siempre tiene el índice como daytime, entonces tengo las horas. Nada más una línea. Sí, nada más una línea. Se me ocurre con para todos. Okay. Ajá. ¿Cómo lo harías? A ver, si fuera una hoja de cálculo, ¿qué haces? saco este los máximos, o sea, saco todo los meses, hago la suma de este consumo energético de todas las horas de un mes, específic de un mes y luego lo repites para todos los meses. Okay. O sea, es eso es lo que haces. ¿Cómo seleccionas todo el mes en pandas?

Hay una función que hace eso. El daytime te ayuda a convertir objetos o a tiempo, pero no hace operaciones. No hay hay dos maneras. Hay una manera muy rebuscada que es un group buy y que es agrupar y entonces agrupas por meses, pero necesitas saber los meses. Hay uno que es poderosísimo, que es el resample.

C e c u e resample nada más. ¿Cómo cómo dices que se llama? C. O cu

promedio. Es que ele es el nombre del data frame, el resample y también, a ver, ¿a qué vo? Eh, quieren hacer series temporales, tienen que estudiar más. Sí, o sea, por ahí en todos los cursos de Python que se dan en el instituto se enseñan esas cosas. de este el resample te permite

su nombre lo dice resample tomar una muestra y entonces primero defines el resample defines el periodo. Entonces tú puedes decir un día, una hora, una semana, un mes, ¿no? Y cuando le dices un mes por default agarra los meses de calendario y luego le dices, ¿qué quieres hacer de ese periodo? el máximo, el mínimo, el promedio, la desviación estándar o la suma, ¿no? Entonces, realmente es una línea, ¿no? Porque si yo le digo

example mes, ahí no me va a dar nada porque me va a decir, "Okay, ¿y qué operación quieres aplicarle?" A pues la suma voy a Ahí está. Y ahí está. Sí.

Y luego viene la otra. Pues esos son unos numerotes, pues lo quiero graficar. Entonces, déjenme hacer un truquito y definir acá la zona de interés, que es igual a esto. Y esto lo voy a nada más para que se vea más bonito. Sí.

O sea, ahí hace lo mismo. ¿Están de acuerdo? Eso me permite escribir este pedacito en función de esto acá y hacerlo más rápido. Sí, aquí yo podría yo puedo mandarlo esto a una variable o lo puedo mandar o puedo usarlo así. La verdad es que está bastante corto. Entonces si yo hago un fix ax

pltots

y otra vez un fix size 12,3. Hasta ahí no estoy haciendo nada nuevo, pero le voy a decir aquí ax plot y voy a graficar esto, pero no quiero hacerlo con un plot, quiero con un

quiero con un con un con un con un box histogram.

No me acuerdo cómo se llama el

plot. Debe haber un box. A ver si hay un box. Ah, box. No, pero no, no es este. Puedo ponerlo en barra horizontal. Barra horizontal en Ah, pues es es de bar. Claro. Eh, el de bar, déjenme quitar esto de aquí porque quiero leer la documentación. El de bar lo que me pide es la posición y luego el valor. Entonces ahí es donde pues otra vez a ver, plotly me sirve. Esto me entregó 12, una lista de 12. Entonces, necesito yo poner las posiciones del 1 al 12. Como yo sé que están ordenadas, ahorita lo puedo revisar, puedo crear un rango para crear las posiciones, pero ahí me está diciendo que necesito X. X es la posición 1 2 3 4 hasta los meses. Entonces puedo crear un range de 1 a 13 para que me cree del 1 al 12. Y luego en el hete es la altura, es el valor de la operación que estoy haciendo y los demás son los valores por default. Entonces yo le puedo decir, "Ah, pues fíjense, fíjense lo que voy a hacer." ¿Vieron? O sea, de repente no hacemos esto, pero ahí le voy a dar un enter y entonces yo puedo escribir ahí más fácil y se ve más bonito. Entonces quiero un range hasta 13 y luego quiero un aa que ese ya lo había sacado. Y es este punto resample. Estoy haciendo lo mismo. Ah, pero espérenme, había dicho que le iba a llamar zona. Perdón, resampo mensual
