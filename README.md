1

Acquisition des données :

// Définition de la broche du potentiomètre
int potPin = A0;

void setup() {
  // Ouvre le port série pour afficher les données
  Serial.begin(9600);
}

void loop() {
  // Lit la valeur du potentiomètre
  int potValue = analogRead(potPin);

  // Affiche la valeur du potentiomètre sur le port série
  Serial.println(potValue);

  // Attend 100 millisecondes avant la prochaine lecture
  delay(100);
}
