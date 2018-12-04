#include <stdio.h>
#include<string.h>
#include<stdlib.h>

typedef struct{
			char meno[51];
			char pohl;
			int rok;
			char spz[8];
			int typ;
			int pokuta;
			long datum;
			} POKUTY;

			
void otvorenie(POKUTY **pole,int *poc){
	FILE *sub;								//otvorenie pomocneho suboru
	sub=fopen("priestupky.txt","r");
	char c;						
	int riadky=0,i;
	
	while (c!=EOF){
		c=fgetc(sub);			//zistovanie poctu riadkov
		if (c=='\n') riadky++;	
	}
	riadky++;  //posledny prazdny riadok
	*poc=riadky/8; 		//zistovanie poctu zaznamov
	*pole=(POKUTY*)malloc(*poc * sizeof(POKUTY));
	fclose(sub);	
	sub=fopen("priestupky.txt","r");		//otvorenie suboru na prvej pozicii
	for (i=0;i<*poc;i++){
		fgets((*pole)[i].meno,100,sub);
		(*pole)[i].meno[strlen((*pole)[i].meno)-1]=0;
		fscanf(sub,"%c",&(*pole)[i].pohl);
		fscanf(sub,"%d",&(*pole)[i].rok);
		fscanf(sub,"%s",(*pole)[i].spz);
		fscanf(sub,"%d",&(*pole)[i].typ);                  //nacitavanie zo suboru
		fscanf(sub,"%d",&(*pole)[i].pokuta);
		fscanf(sub,"%li",&(*pole)[i].datum);
		fgetc(sub);fgetc(sub);
	}
	fclose(sub);
}


void vypis(POKUTY *pole,int poc){
	int i;
	for(i=0;i<poc;i++){
		printf("Cele meno: %s\n",pole[i].meno);
		printf("Pohlavie: %c\n",pole[i].pohl);
		printf("Rok nar.: %d\n",pole[i].rok);				//vypis jednotlivych poloziek
		printf("SPZ: %s\n",pole[i].spz);
		printf("Typ pokuty: %d\n",pole[i].typ);
		printf("Vyska pokuty: %d\n",pole[i].pokuta);
		printf("Datum: %li\n\n",pole[i].datum);
	}
}


void palindrom(POKUTY *pole,int poc){
	int i;
	for(i=0;i<poc;i++){
		if ((pole[i].spz[0]==pole[i].spz[6])&&(pole[i].spz[1]==pole[i].spz[5])&&(pole[i].spz[2]==pole[i].spz[4])){         		//vypis palindromov
			printf("Cele meno: %s %s \n",pole[i].meno,pole[i].spz);
		}
	}
}


void odmena(POKUTY *pole,int poc){
	int i;
	long d;
	double vys=0;
	printf("Zadaj datum v tvare RRRRMMDD: \n");		
	scanf("%li",&d);
	for(i=0;i<poc;i++){
		if ((pole[i].datum/100)==(d/100)){		//vyhladavanie priestupkov v danom mesiaci
			if (pole[i].typ=='1')
				vys+=pole[i].pokuta*0.052;
			else 
				vys+=pole[i].pokuta*0.038;	// pripocitavanie vysky odmeny
			}
	}
if (vys!=0) 
	printf("%.2lf",vys);	//vypis odmeny
}


void odnatie(POKUTY *pole,int poc){
	long d;
	int i;
	printf("Zadaj datum v tvare RRRRMMDD: \n");
	scanf("%li",&d);
	for(i=0;i<poc;i++){
		if (((d/10000)==(pole[i].datum/10000))&&(pole[i].typ==0)){		//vyhladavanie vaznych pristupkov
				printf("Cele meno: %s \n",pole[i].meno);
				printf("SPZ: %s \n",pole[i].spz);
				printf("Datum: %li \n \n",pole[i].datum);
		}
	}	
}


void koniec(POKUTY **pole){
	free(*pole);		//dealokovanie pola
	exit(1);		//ukoncenie programu
}


void pridaj(POKUTY **pole,int *poc){
	int i;
	getchar();
	if (*pole==NULL){			
		*pole=(POKUTY*)malloc(sizeof(POKUTY));
		gets((*pole)[0].meno);
		scanf("%c",&(*pole)[0].pohl);
		scanf("%d",&(*pole)[0].rok);
		scanf("%s",(*pole)[0].spz);
		scanf("%d",&(*pole)[0].typ);                  //nacitavanie z klavesnice na nulovu poziciu, v pripade ze uzivatel stlaci 'a' pred 'o'
		scanf("%d",&(*pole)[0].pokuta);
		scanf("%li",&(*pole)[0].datum);
		*poc = 1;
		return;
		}
	else {
		*pole=(POKUTY*)realloc(*pole,(++*poc)*sizeof(POKUTY));			//realokovanie pola
		gets((*pole)[*poc-1].meno);
		scanf("%c",&(*pole)[*poc-1].pohl);
		scanf("%d",&(*pole)[*poc-1].rok);
		scanf("%s",(*pole)[*poc-1].spz);
		scanf("%d",&(*pole)[*poc-1].typ);                  
		scanf("%d",&(*pole)[*poc-1].pokuta);
		scanf("%li",&(*pole)[*poc-1].datum);
		}
	POKUTY osoba;
	for (i=*poc-1;i>=0;i--){								//vlozenie pridaneho zaznamu podla abecedy
		if (strcmp((*pole)[i].meno,(*pole)[i-1].meno)){
			osoba=(*pole)[i];							
			(*pole[i])=(*pole)[i-1];
			(*pole)[i-1]=osoba;
		}
	}
}


void mazanie(POKUTY **pole,int *poc){
	int i,pom=0,x=0;
	long d;
	printf("Zadaj datum v tvare RRRRMMDD: \n");
	scanf("%li",&d);
	for (i=0;i<*poc;i++){
		if ((d>(*pole)[i].datum)&&((*pole)[i].typ=='0')){
			++x;
			free(*pole[i]);
		}
		else ++pom;
	}
	if(pom==(*poc))
		printf("Zaznamy neboli zistene");
	else 
		*pole=(POKUTY*)realloc(*pole,(*poc-x) * sizeof(POKUTY));
}


int main(){
	POKUTY *pole = NULL;
	char c;
	int poc=0;				
	while(true){
		scanf("%c",&c);			//nacitavanie znakov
		if (c=='o')
			otvorenie(&pole,&poc); 
		if (c=='v') 
			vypis(pole,poc);
		if (c=='p') 
			palindrom(pole,poc);
		if (c=='r') 
			odmena(pole,poc);
		if (c=='x')
			odnatie(pole,poc);
		if (c=='a')
			pridaj(&pole,&poc); 
		if (c=='k') 
			koniec(&pole);
		if (c=='m')
			mazanie(&pole,&poc);
		//if (c=='s')
			//cezar(pole,poc);
	}
	return 0;
}

			
