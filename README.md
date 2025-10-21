# Programación C
![Logo de C](https://upload.wikimedia.org/wikipedia/commons/1/19/C_Logo.png)
## Descripción
## Este ejercicio es mi práctica final de C, el ejercicio consiste en crear tres procesos: 2 hijos y un padre. 
El proceso padre genera 5 expresiones aleatorias, se las envia al hijo1 para que las calcule, el padre recibe el resultado de la operación y se lo envia al hijo2 para que confirme si la operacion es correcta o no
---
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <fcntl.h>
#include <unistd.h>
#include <time.h>
#include <signal.h>

#define SIZE 20
//Indica el numero de operaciones que tienen que aparecer en la tabla
#define INTENTOS 5


//Funciones prototipo
void codigo_hijo1();
void codigo_hijo2();
void codigo_padre();


//variables globales
int tP1H[2];
int tH1P[2];
int tP2H[2];
int tH2P[2];

int main(void)
{
  pid_t pid1;
  pid_t pid2;
  pipe(tP1H);
  pipe(tH1P);
  pipe(tP2H);
  pipe(tH2P);
  srand(time(NULL));
  pid1 = fork();
  switch(pid1)
  {
     case -1:printf("El proceso hijo1 no ha podido crearse\n");
	    exit(-1);
	    break;
     case 0:codigo_hijo1();
	    exit(0);
	    break;
     default:
	 pid2 = fork();
	 switch(pid2)
 	 {
	   case -1: printf("El proceso hijo2 no ha podido crearse\n");
	   	    exit(-1);
		    break;
	   case 0:codigo_hijo2();
		    exit(0);
		    break;
	   default: codigo_padre();
		    break;
	 }
    }
    wait(NULL);
    wait(NULL);
    return 0;

  }
  //El hijo1 recibe la expresión del padre,la calcula y se la enviará al padre 
  void codigo_hijo1()
  {
    close(tP1H[1]);
    close(tH1P[0]);
    for(int i=0;i<INTENTOS;i++)
    {
      char expresion[SIZE];
      int num1, num2, resultado;
      char op;
      read(tP1H[0],expresion,20);
      sscanf(expresion,"%d %c %d", &num1, &op, &num2);
      switch(op)
      {
        case '+': resultado = num1 + num2;
		break;
        case '-': resultado = num1 - num2;
		break;
        case '*': resultado = num1 * num2;
		break;
	case '/': resultado = num1 / num2;
		break;
      }
     if(rand()%5==0)
	resultado +=1;
     write(tH1P[1], &resultado,4);

    }
    close(tP1H[0]);
    close(tH1P[1]);
  }
  //El hijo2 recibe del padre la expresión y el resultado que ha generado, el hijo1 comprueba si la  operacioń se ha realizado correctamente
  void codigo_hijo2()
  {
    close(tP2H[1]);
    close(tH2P[0]);
    for(int i=0;i<INTENTOS;i++)
    {
      char expresion[SIZE];
      int resultadoH1;
      read(tP2H[0],expresion,20);
      read(tP2H[0],&resultadoH1,4);
      int num1, num2, resultadoFinal;
      char op;
      sscanf(expresion,"%d %c %d",&num1, &op, &num2);
      switch(op)
      {
        case '+':resultadoFinal = num1 + num2;
		break;
        case '-':resultadoFinal = num1 - num2;
		break;
        case '*':resultadoFinal = num1 * num2;
		break;
	case '/':resultadoFinal = num1 / num2;
		break;
      }
     char confirmacion[12];
     if(resultadoH1 == resultadoFinal)
	strcpy(confirmacion, "correcto");
     else
	strcpy(confirmacion, "incorrecto");
     write(tH2P[1],&resultadoFinal,4);
     write(tH2P[1],confirmacion,12);
    }
   close(tP2H[0]);
   close(tH2P[1]);
  }
  //El padre se encarga de generar la expresión, de enviarla a los hijos y recibir la expresión corregida
  void codigo_padre(char expresion[])
  {
    close(tP1H[0]);
    close(tH1P[1]);
    close(tP2H[0]);
    close(tH2P[1]);
    printf("--------------------------------------------------------\n");
    printf("Nº | Operación | ResultadoH1 | Confirmación | ResultadoH2\n");
    printf("--------------------------------------------------------\n");
    for(int i=0;i<INTENTOS;i++)
    {
      int numero1,numero2;
      int resultadoH1, resultadoH2;
      char operador;
      char confirmacion[12];
      char operadores[]={'+','-','*','/'};
      char expresion[SIZE];
      
      numero1 = rand() % 100 +1;
      numero2 = rand() % 100 +1;
      operador = operadores[rand() % 4];
      if(operador == '/' && numero2 == 0)
	 numero2 = 1;
      sprintf(expresion,"%d %c %d",numero1,operador,numero2);

      //Enviar la operacion al hijo1
      write(tP1H[1],expresion,20);
      //Leer el resultado de la operacion de hijo1
      read(tH1P[0],&resultadoH1,4);
      //Envia la operación y el resultado al hijo2 para que la confirme
      write(tP2H[1],expresion,20);
      write(tP2H[1],&resultadoH1,4);
      //Leer la respuesta del hijo2
      read(tH2P[0],&resultadoH2,4);
      read(tH2P[0],confirmacion,12);
      printf("%d  | %-9s | %-11d | %-12s | %-11d\n",i+1,expresion,resultadoH1,confirmacion,resultadoH2);
    }
   close(tP1H[1]);
   close(tH1P[0]);
   close(tP2H[1]);
   close(tH2P[0]);
  }
````
---
