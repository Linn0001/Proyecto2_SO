# AL02___Dummy_Syscall

Sistemas Operativos CS3015 2026-1
Actividad de Laboratorio 2: Creando nuestra primera syscall en Selfie
Profesores Jorge Gonzalez, Mauricio Pinto
Colaboradores Mariana Capu˜ nay
Contexto
En un sistema operativo, es com´ un que los programas de usuario necesiten acceder a ciertos recursos del sistema
como buffers, archivos o memoria. Estos recursos requieren interacci´ on delkernel, y son lassyscallslas cu´ ales
proveen el acceso a sus servicios. Esta syscall provoca una excepci´ on en la ejecuci´ on del programa, la cual fuerza
un cambio de contexto (de usuario akernel) para poder acceder a los privilegios necesarios para llevarla a cabo.
Luego, mediante unhandlerse mapea la syscall que debe ser ejecutada, se realiza la operaci´ on y se retorna al
contexto de usuario.
Objetivos
1. Comprender el funcionamiento de una syscall
2. Comprender c´ omo Selfie maneja las excepciones
3. Recibir argumentos desde una syscall en Selfie
4. Retornar un valor desde una syscall en Selfie
Funcionamiento
El c´ odigo mostrado define una funci´ onmainque invoca la syscalldummy syscallcon un argumento de 20 y
almacena el valor retornado enx. Posteriormente, el resultado es impreso utilizando la funci´ onprint number.
int main(int argc, char** argv) –
uint64˙t x;
x = dummy˙syscall(20);
print˙number(x);
˝
El objetivo es quedummy syscallpueda sumar 2026 al valor de entrada, la ejecuci´ on del c´ odigo deber´ a generar
la siguiente salida en consola:
2045
La funci´ onprint numberconvierte un n´ umero entero en su representaci´ on ASCII e imprime cada d ´ ıgito in-
dividualmente en la consola utilizando la syscallwrite. Para ello, descompone el n´ umero en sus d ´ ıgitos m´ as
significativos y los escribe uno por uno. Esta funci´ on es necesaria porqueselfieno tiene un m´ etodo nativo para
imprimir n´ umeros enteros de m´ as de un d ´ ıgito.
void print˙number(uint64˙t number) –
uint64˙t digit;
uint64˙t divisor;
uint64˙t *d;
d = malloc(sizeof(uint64˙t));
divisor = 1;
Abril 2026


while (number / divisor ¿= 10) –
divisor = divisor * 10;
˝
while (divisor ¿ 0) –
digit = number / divisor;// most significant digit
number = number % divisor;// less significant digits
divisor = divisor / 10;
*d = 48 + digit;// 48 = ASCII('0')
write(1, d, 1);// write the digit in console
˝
˝
Desarrollo
1. Syscall ID
Dentro de las contantes deselfie, se encuentra una serie de valores (IDs) las cuales permiten mantener un mapeo
ordenado de las syscalls. Esto sirve para que, luego de detectar un interrupt,selfiepueda saber qu´ e syscall gener´ o
la excepci´ on.
Para completar este ´ ıtem, debes a˜ nadir un ID para la nueva dummy syscall.
uint64˙t SYSCALL˙EXIT = 93;
uint64˙t SYSCALL˙READ = 63;
uint64˙t SYSCALL˙WRITE = 64;
uint64˙t SYSCALL˙OPENAT = 56;
uint64˙t SYSCALL˙BRK = 214;
uint64˙t SYSCALL˙DUMMY˙SYSCALL = 01;// nuevo ID para nuestra syscall
Cuidado
Si repites un ID,selfiepuede manejar tu syscall como si fuese otra, lo cual podria llevar a un undefined
behaviour, y a pasar horas depurando!
2. handle exception
Para manejar excepciones, selfie cuenta con la funci´ on:
uint64˙t handle˙exception(uint64˙t *context);
En caso la excepci´ on haya sido generada por una syscall, se ingresa a la funci´ on encargada de verificar y gestionar
la syscall que ha sido llamada en tiempo de ejecuci´ on:
uint64˙t handle˙system˙call(uint64˙t *context);
Dentro de esta ´ ultima funci´ on tambi´ en debe gestionarse el llamado de nuestradummysyscall.
Para completar este ´ ıtem, debes manejar la excepci´ on originada por la dummy syscall, dentro de han-
dle system call.
3. emit dummy syscall
Para la creaci´ on de una syscall en selfie, se debe definir una funci´ on que sirva como emit wrapper. En ella, debemos
indicar la palabra a trav´ es de la cual se har´ a el llamado a nuestra syscall, as ´ ı como tambi´ en la cantidad de argumentos
que recibe, su tipo de retorno y los registros en los que almacenaremos ambos.
Para completar este ´ ıtem, debes a˜ nadir el emit wrapper de la nueva dummy syscall en selfie.
2

void emit˙dummy˙syscall()
–
create˙symbol˙table˙entry(GLOBAL˙TABLE, string˙copy(”dummy˙syscall”),
0, PROCEDURE, UINT64˙T, 1, code˙size);
emit˙load(REG˙A0, REG˙SP, 0);// value
emit˙addi(REG˙SP, REG˙SP, WORDSIZE);
emit˙addi(REG˙A7, REG˙ZR, SYSCALL˙DUMMY˙SYSCALL);
emit˙ecall();
emit˙jalr(REG˙ZR, REG˙RA, 0);
˝
Hint
Considere que lasyscall writese llama a trav´ es de la palabra”write”, recibe 3 argumentos (fd,
*buffer,size), los cuales se escriben en los registros: REG A0, REG A1, REG A2, respectivamente; y
retorna un valor de tipouint64 ten el registro REG 0.
La syscall dummy debe recibir un argumento y retornar un valor de tipo uint64 t.
Cuidado
No olvide llamar a la funci´ onemitde esta nueva syscall en la funci´ onselfie compile(); si no lo hace,
selfieno detectar´ a la syscall y marcar´ a un error.
4. do ecall
La funci´ on encargada de alterar y leer el valor de los registros es:
void do˙ecall()
Dentro de esta funci´ on se gestionan los registros usados por cada syscall. Para el caso de dummy syscall,
usaremos el registro REG A0, tanto para leer el par´ ametro como para retornar la suma.
else –
read˙register(REG˙A0);
// aqu´ ı comenzar´ ıa el listado de nuevas syscalls
if (*(registers + REG˙A7) == SYSCALL˙DUMMY˙SYSCALL)
write˙register(REG˙A0);
// ¡¡ todas las nuevas sycalls que creemos deben ser listadas aqu´ ı ¿¿
else// implementaciones ya creadas por selfie
–
// c´ odigo ya existente en selfie...
˝
// all system calls other than switch are handled by exception
throw˙exception(EXCEPTION˙SYSCALL, a7);
˝
Para completar este ´ ıtem, debes indicar qu´ e registros necesita gestionar nuestra dummy syscall en doecall.
Hint
Los m´ etodos para escribir y leer un registro son:read register(REG Ai)ywrite register(REG Ai),
respectivamente.
3

5. implement dummy syscall
Para implementar el funcionamiento de nuestra syscall, debemos crear una funci´ on que lea y escriba los registros
definidos en la funci´ onemitde nuestra syscall.
void implement˙dummy˙syscall(uint64˙t *context)
Para finalizar este laboratorio, debes implementar el funcionamiento de la nueva dummy syscall dentro
de la funci´ on implementdummy syscall.
Hint
Considere que la implementaci´ on de lasyscall writese encarga de leer los 3 argumentos que recibe y
de escribir su valor de retorno.
Recuerde que nuestradummy syscallespera un valor y retorna ese mismo valor, sum´ andole 2026.
Cuidado
•No olvide incrementar elprogram counter; de lo contrario, el programa se quedar´ a en la misma l ´ ınea
de manera indeterminada.
•En selfie, los valores de retorno se guardan en registros, por lo que hacerreturn value + 2026;no
funcionar´ a. En su lugar, intente escribirlo usando:*(get regs(context) + REG A0)para referenciar
el valor de retorno (especificado en eldo ecall).
4