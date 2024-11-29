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
Acquisition Visualisation IHM: 
function Acquisition8
    % Créer la figure pour l'interface IHM
    fig = figure('Name', 'Interface de contrôle', 'NumberTitle', 'off', 'Position', [500, 500, 400, 300]);

    % Label et champ de texte pour le nombre d'essais
    uicontrol('Style', 'text', 'Position', [50, 240, 300, 30], 'String', 'Nombre d''essais :', 'FontSize', 12);
    numTrialsEdit = uicontrol('Style', 'edit', 'Position', [170, 220, 50, 25], 'String', '3', 'FontSize', 12);

    % Label et champ de texte pour la durée des essais
    uicontrol('Style', 'text', 'Position', [50, 180, 300, 30], 'String', 'Durée des essais (s) :', 'FontSize', 12);
    trialDurationEdit = uicontrol('Style', 'edit', 'Position', [170, 160, 50, 25], 'String', '5', 'FontSize', 12);

    % Label et champ de texte pour le chemin d'enregistrement des fichiers
    uicontrol('Style', 'text', 'Position', [50, 120, 300, 30], 'String', 'Répertoire de sauvegarde des fichiers :', 'FontSize', 12);
    folderPathEdit = uicontrol('Style', 'edit', 'Position', [50, 80, 300, 30], 'String', 'C:\Data', 'FontSize', 12);

    % Bouton de démarrage
    startButton = uicontrol('Style', 'pushbutton', 'Position', [100, 20, 200, 40], 'String', 'Démarrer les essais', 'FontSize', 12,'BackgroundColor', 'g');

    % Callback du bouton de démarrage
    set(startButton, 'Callback', @(src, event) startTrial(numTrialsEdit, trialDurationEdit, folderPathEdit, fig));
end

function startTrial(numTrialsEdit, trialDurationEdit, folderPathEdit, fig)
    % Récupérer les paramètres de l'IHM
    totalTrials = str2double(numTrialsEdit.String);
    trialDuration = str2double(trialDurationEdit.String);
    folderPath = folderPathEdit.String;

    % Vérification si les paramètres sont valides
    if isempty(totalTrials) || totalTrials <= 0
        errordlg('Le nombre d''essais doit être un nombre entier positif.', 'Erreur');
        return;
    end
    if isempty(trialDuration) || trialDuration <= 0
        errordlg('La durée des essais doit être un nombre entier positif.', 'Erreur');
        return;
    end
    if isempty(folderPath) || ~isfolder(folderPath)
        errordlg('Le chemin spécifié pour l''enregistrement des fichiers est invalide.', 'Erreur');
        return;
    end

    % Fermer la fenêtre IHM
    close(fig);

    % Définir le port COM de ton Arduino (par exemple COM7)
    port = "COM7";  % Remplace "COM7" par le port de ton Arduino
    baudRate = 9600;  % Doit correspondre à la vitesse de transmission définie dans Arduino (Serial.begin(9600))

    % Ouvre la connexion série
    s = serialport(port, baudRate);

    % Configurer le timeout (en secondes)
    s.Timeout = 10;  % Délai d'attente de 10 secondes pour recevoir des données

    % Variables pour stocker les résultats
    rawData = {};  % Cell array pour stocker les données brutes
    voltageData = {};  % Cell array pour stocker les tensions
    timestamps = {};  % Cell array pour stocker les temps de lecture

    % Boucle pour effectuer les essais
    for trial = 1:totalTrials
        disp(['Début de l''essai ', num2str(trial), '...']);
        disp(['Appuyez sur une touche pour démarrer l''essai ', num2str(trial), '.']);
        pause;  % Attendre l'action de l'utilisateur pour chaque essai

        startTime = tic;  % Démarrer le chronomètre pour l'essai
        trialRawData = [];  % Temporaire pour les données brutes
        trialVoltageData = [];  % Temporaire pour les tensions
        trialTimestamps = [];  % Temporaire pour les temps

        % Boucle pour enregistrer les données pendant la durée de l'essai
        while toc(startTime) < trialDuration  % Limiter l'essai à 5 secondes
            % Lire une ligne de données envoyée par Arduino
            data = readline(s);
            value = str2double(data);  % Convertir la valeur brute en nombre
            voltage = (value / 1023) * 5;  % Calculer la tension

            % Ajouter les données à la liste
            trialRawData = [trialRawData, value];
            trialVoltageData = [trialVoltageData, voltage];
            trialTimestamps = [trialTimestamps, toc(startTime)];  % Enregistrer le temps écoulé

            % Affichage en continu dans la fenêtre de commande
            disp(['Temps: ', num2str(toc(startTime), '%.3f'), 's, Valeur brute: ', num2str(value), ', Tension (V): ', num2str(voltage, '%.3f')]);

            % Pause pour éviter une lecture trop rapide
            pause(0.1);
        end

        % Ajouter les résultats de l'essai actuel aux données totales
        rawData{trial} = trialRawData;
        voltageData{trial} = trialVoltageData;
        timestamps{trial} = trialTimestamps;

        % Enregistrer les données de l'essai dans un fichier .txt
        fileName = fullfile(folderPath, ['essai', num2str(trial), '.txt']);  % Créer un nom de fichier unique
        fid = fopen(fileName, 'w');  % Ouvrir le fichier en mode écriture
        if fid == -1
            disp('Erreur lors de l''ouverture du fichier');
        else
            fprintf(fid, 'Temps (s)\tValeur brute\tTension (V)\n');  % En-têtes du fichier
            for i = 1:length(trialRawData)
                fprintf(fid, '%.3f\t%d\t%.3f\n', trialTimestamps(i), trialRawData(i), trialVoltageData(i));  % Écrire les données dans le fichier
            end
            fclose(fid);  % Fermer le fichier après écriture
            disp(['Données enregistrées dans ', fileName]);
        end
    end

    % Visualisation des résultats après tous les essais
    if totalTrials > 6
        nCols = 2;  % Deux colonnes
        nRows = ceil(totalTrials / 2);  % Nombre de lignes (en divisant les essais par deux)
    else
        nCols = 1;  % Une seule colonne
        nRows = totalTrials;  % Nombre de lignes égale au nombre d'essais
    end

    % Graphique pour les valeurs brutes
    figure;
    for trial = 1:totalTrials
        subplot(nRows, nCols, trial);  % Créer un sous-graphique pour chaque essai
        plot(timestamps{trial}, rawData{trial}, 'b-', 'DisplayName', 'Valeur brute');
        xlabel('Temps (s)');
        ylabel('Valeur brute');
        title(['Essai ', num2str(trial), ' - Evolution des valeurs brutes du potentiomètre']);
        ylim([100 650]);  % Fixer l'échelle de l'axe Y entre 100 et 650
        legend;
        grid on;
    end

    % Graphique pour les tensions
    figure;
    for trial = 1:totalTrials
        subplot(nRows, nCols, trial);  % Créer un sous-graphique pour chaque essai
        plot(timestamps{trial}, voltageData{trial}, 'r-', 'DisplayName', 'Tension (V)');
        xlabel('Temps (s)');
        ylabel('Tension (V)');
        title(['Essai ', num2str(trial), ' - Evolution de la tension du potentiomètre']);
        ylim([0 4]);  % Fixer l'échelle de l'axe Y entre 0.5 et 2.5 V
        legend;
        grid on;
    end

    % Fin du programme
    disp('Tous les essais sont terminés.');
end
