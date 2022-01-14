

#define IRon 11 //çizgi sensörleri
#define IRarka 12

#define MotorR1 6 //motor çıkışları
#define MotorR2 7
#define MotorRE 5

#define MotorL1 8
#define MotorL2 9
#define MotorLE 10

#define sensorsag 2 // MZ-80 sensörler
#define sensororta 3
#define sensorsol 4

///gerekli bazı değişkenleri tanımlayalım
 int distance_on;
 int distance_sag;
 int distance_sol;
 int IR_on;
 int IR_arka;
 int deger;


//setup kodu bir defa çalışacak
void setup() {
  //pin tanımlamalarını yapalım
 // 2 adet çizgi tespit sensörü
  pinMode(IRon, INPUT);
  pinMode(IRarka, INPUT);
  
  pinMode(sensorsag, INPUT); // önde, solda ve sağda olmak üzere 3 adet cisim sensörü
  pinMode(sensororta, INPUT);
  pinMode(sensorsol, INPUT);

  
  pinMode(MotorR1, OUTPUT); //motor çıkış pinleri
  pinMode(MotorR2, OUTPUT);
  pinMode(MotorL1, OUTPUT);
  pinMode(MotorL2, OUTPUT);
  
  Serial.begin(9600);

}

//ana gövde devamlı çalışıyor
void loop() {

 IR_on = digitalRead(IRon); //çizgi sensörlerinden veri okuyoruz
 IR_arka = digitalRead(IRarka);

 while (IR_on == 0 && IR_arka == 0) {  //rakip tespit etmeyi bu while içinde yapıcam. eğer çizgi sensörlerden herhangi birinde çizgi sinyali almışsa
  //zaten öncelik kendini kurtarmak olacak ve bu while içine hiç girmeyecek
   Serial.println("çizgiye basmıyorum ringin içindeyim cisim aramaya başla"); 

    distance_on=digitalRead(sensororta); //ön sensörden ölçüm alıyoruz
    distance_sag=digitalRead(sensorsag); //sag sensörden ölçüm alıyoruz
    distance_sol=digitalRead(sensorsol); //sol sensörden ölçüm alıyoruz
    
   //MZ-80 sensörler cisim görünce 0 çıkışı veriyor !!!!!!!!!!  
   while (distance_on == 0) { //ön sensörde cisim algılamışsa
          //önce çizgiye basıp basmadığını kontrol edelim. Basıyorsa hiçbirşey yapmayıp döngüden çıkacak
         IR_on = digitalRead(IRon);
         IR_arka = digitalRead(IRarka);
          if (IR_on == 1 || IR_arka == 1){
            break;
            
            }

          //eğer çizgiye basmıyorsa önde algılama olduğu için rakibe saldıracak  
         saldir();
         
         Serial.println("hedefe git");
          distance_on=digitalRead(sensororta);//while da sonsuz dönmesin diye tekrar ön sensörde algılama olup olmadığını kontrol
          //ediyoruz 

    } //while

        
    while ((distance_sag == 0 && distance_on ==1 )){ // sadece sağdan bilgi alıyor sağa dönecek
         IR_on = digitalRead(IRon); //çizgi kontrolü
         IR_arka = digitalRead(IRarka);
          if (IR_on == 1 || IR_arka == 1){
            break; //çizgi varsa çık
            
            }

            yerindesag(); //çizgi yok sağa dönecek
            delay(200);   //bu değeri aracınızın 90 derece dönmesini sağlayacak şekilde deneme yanılma ile bulun  
            Serial.println("sağa dön");
        distance_on=digitalRead(sensororta); //ön sensörden ölçüm alıyoruz ki whileda kalmasın
        distance_sag=digitalRead(sensorsag); //sag sensörden ölçüm alıyoruz
        distance_sol=digitalRead(sensorsol); //sol sensörden ölçüm alıyoruz 
    
        }

   while ((distance_sol == 0 && distance_on ==1 )){ // sadece soldan bilgi alıyor sola dönecek sağ kısmının aynısı
         IR_on = digitalRead(IRon);
         IR_arka = digitalRead(IRarka);
          if (IR_on == 1 || IR_arka == 1){
            break;
            
            }
            yerindesol();
            delay(200);      
        Serial.println("sola dön");
        distance_on=digitalRead(sensororta); //ön sensörden ölçüm alıyoruz
        distance_sag=digitalRead(sensorsag); //sag sensörden ölçüm alıyoruz
        distance_sol=digitalRead(sensorsol); //sol sensörden ölçüm alıyoruz 
    
        }


        
    while ((distance_sag ==1)&&(distance_on == 1) && (distance_sol == 1)){ //hiçbir sensörde cisim yok ne yapalım?
         IR_on = digitalRead(IRon); //çizgileri kontrol et
         IR_arka = digitalRead(IRarka);
          if (IR_on == 1 || IR_arka == 1){
            break; //çizgiye basıyorsan çık
            
            }
          ileri(); // cisim görmediğinde yapılacak şey size kalmış biz düz ileri gitmesini istedik.
          //yerinde dönme de yaptırılabilir.
          
         Serial.println("cisim bulunamadı ileri git");
         distance_on=digitalRead(sensororta); //ön sensörden ölçüm alıyoruz ki while da kalmasın
        distance_sag=digitalRead(sensorsag); //sag sensörden ölçüm alıyoruz
        distance_sol=digitalRead(sensorsol); //sol sensörden ölçüm alıyoruz 
    
    }  
          
       
         IR_on = digitalRead(IRon); //çizgi kontrolü yapalım ki ilk while da sonsuz dönmesin
         IR_arka = digitalRead(IRarka);
 }//ilk while çıkış

  
  if (IR_on== 1){  //ön çizgi sensörü çizgi algılıyorsa geri gidecek
  Serial.println(" ön çizgi tespit edildi geri gidiyorum"); 
      geri();
      delay(400); // delay değerleri yine ringinizin boyutuna ve aracınızın hızına göre deneme ile ayarlanabilir.
      yerindesag();
      delay(300); //// delay değerleri yine ringinizin boyutuna ve aracınızın hızına göre deneme ile ayarlanabilir.

 }   
 else if (IR_arka == 1){
 // else if (IR_arka == 1 && (IR_sol ==0 && IR_sag==0)){ //sadece arka sensör gördü o zaman ileri gitmeli
   Serial.println("arka çizgi tespit edildi ileri gidiyorum");
   ileri();
   delay(400);//// delay değerleri yine ringinizin boyutuna ve aracınızın hızına göre deneme ile ayarlanabilir.

  }   


} //loop sonu

void saldir() { // Robotun saldırma hareketi için fonksiyon tanımlıyoruz.

  digitalWrite(MotorR1, HIGH); // Sağ motorun ileri hareketi aktif
  digitalWrite(MotorR2, LOW); // Sağ motorun geri hareketi pasif
  analogWrite(MotorRE, 220); // Sağ motorun hızı 

  digitalWrite(MotorL1, HIGH); // Sol motorun ileri hareketi aktif
  digitalWrite(MotorL2, LOW); // Sol motorun geri hareketi pasif
  analogWrite(MotorLE, 220); // Sol motorun hızı 


}

void ileri() { // Robotun ileri yönde hareketi için fonksiyon tanımlıyoruz.

  digitalWrite(MotorR1, HIGH); // Sağ motorun ileri hareketi aktif
  digitalWrite(MotorR2, LOW); // Sağ motorun geri hareketi pasif
  analogWrite(MotorRE, 130); // Sağ motorun hızı

  digitalWrite(MotorL1, HIGH); // Sol motorun ileri hareketi aktif
  digitalWrite(MotorL2, LOW); // Sol motorun geri hareketi pasif
  analogWrite(MotorLE, 130); // Sol motorun hızı 


}


void yerindesag() { // Robotun sağa dönme hareketi için fonksiyon tanımlıyoruz.

  digitalWrite(MotorR1, LOW); // Sağ motorun ileri hareketi pasif
  digitalWrite(MotorR2, HIGH); // Sağ motorun geri hareketi aktif
  analogWrite(MotorRE, 180); // Sağ motorun hızı 

  digitalWrite(MotorL1, HIGH); // Sol motorun ileri hareketi aktif
  digitalWrite(MotorL2, LOW); // Sol motorun geri hareketi pasif
  analogWrite(MotorLE, 180); // Sol motorun hızı 


}
void yerindesol() { // Robotun sola dönme hareketi için fonksiyon tanımlıyoruz.

  digitalWrite(MotorR1, HIGH); // Sağ motorun ileri hareketi aktif
  digitalWrite(MotorR2, LOW); // Sağ motorun geri hareketi pasif
  analogWrite(MotorRE, 180); // Sağ motorun hızı 

  digitalWrite(MotorL1, LOW); // Sol motorun ileri hareketi pasif
  digitalWrite(MotorL2, HIGH); // Sol motorun geri hareketi aktif
  analogWrite(MotorLE, 180); // Sol motorun hızı 
}


void dur() { // Robotun durma hareketi için fonksiyon tanımlıyoruz.

  digitalWrite(MotorR1, HIGH);
  digitalWrite(MotorR2, LOW);
  digitalWrite(MotorRE, LOW);

  digitalWrite(MotorL1, HIGH);
  digitalWrite(MotorL2, LOW);
  digitalWrite(MotorLE, LOW);

}
void geri() { // Robotun geri hareketi için fonksiyon tanımlıyoruz.

  digitalWrite(MotorR1, LOW); // Sağ motorun ileri hareketi pasif
  digitalWrite(MotorR2, HIGH); // Sağ motorun geri hareketi aktif
  analogWrite(MotorRE, 130); // Sağ motorun hızı 

  digitalWrite(MotorL1, LOW); // Sol motorun ileri hareketi pasif
  digitalWrite(MotorL2, HIGH); // Sol motorun geri hareketi aktif
  analogWrite(MotorLE, 130); // Sol motorun hızı 

}
