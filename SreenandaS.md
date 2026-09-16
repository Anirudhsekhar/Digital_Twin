%% Landslide Digital Twin — Data Collection & Function Generation
% My part of the team project: data collection + function generation.
% Save as .mlx via File > Save As > MATLAB Live Code Files (*.mlx)

clear; clc; close all;

numDays = 30;

%% Data Collection - Rainfall
rainfallTable = generateSyntheticRainfall(numDays, 42);
disp(rainfallTable)

%% Data Collection - Terrain
terrain = generateSyntheticTerrain(1);
disp(terrain)

%% Data Collection - Soil Moisture
soilMoisture = generateSyntheticSoilMoisture(numDays, rainfallTable);
plot(soilMoisture.SoilMoisture_pct)
title('Soil Moisture Over Time')
xlabel('Day'); ylabel('Soil Moisture (%)')

%% Data Collection - Historical Landslide Events
catalog = loadLandslideCatalog('landslide_catalog.csv');
disp(head(catalog, 10))
fprintf('Loaded %d historical landslide records.\n', height(catalog))

%% Function Generation - Slope Stability (Factor of Safety)
testFS = computeSlopeStability(30, 15, 25); % sanity check with plain numbers
disp(testFS) % expect something like 0.8-2.5, not NaN or Inf

FS = computeSlopeStability(terrain.SlopeAngle_deg, terrain.SoilCohesion_kPa, soilMoisture.SoilMoisture_pct);
disp(table(soilMoisture.Date, FS, 'VariableNames', {'Date','FactorOfSafety'}))

%% Function Generation - Rainfall Risk Factor
rainfallRisk = computeRainfallRiskFactor(rainfallTable, 3);

%% Function Generation - Combined Risk Score
[riskScore, riskLevel] = computeLandslideRiskScore(FS, rainfallRisk);

%% Final Output - handoff table for the team
results = table(rainfallTable.Date, rainfallTable.Rainfall_mm, soilMoisture.SoilMoisture_pct, ...
    FS, riskScore, riskLevel, ...
    'VariableNames', {'Date','Rainfall_mm','SoilMoisture_pct','FactorOfSafety','RiskScore','RiskLevel'});
disp(results)
writetable(results, 'landslide_risk_output.csv');
fprintf('Saved landslide_risk_output.csv with %d rows.\n', height(results));


%% ===================== LOCAL FUNCTIONS - DO NOT ADD CODE BELOW THIS =====================

function rainfallTable = generateSyntheticRainfall(numDays, seed)
    rng(seed);
    startDate = datetime('today') - days(numDays-1);
    dates = (startDate:datetime('today'))';

    baseRain = abs(5*randn(numDays,1));
    burstMask = rand(numDays,1) < 0.15; % 15% of days are "storm days"
    burstRain = burstMask .* (40 + 30*rand(numDays,1));
    rainfall = round(baseRain + burstRain, 1);

    rainfallTable = table(dates, rainfall, 'VariableNames', {'Date','Rainfall_mm'});
end

function terrain = generateSyntheticTerrain(numPoints)
    slopeAngle = 15 + 35*rand(numPoints,1); % 15-50 degrees
    cohesion = 5 + 20*rand(numPoints,1);    % kPa
    terrain = table(slopeAngle, cohesion, 'VariableNames', {'SlopeAngle_deg','SoilCohesion_kPa'});
end

function soilMoisture = generateSyntheticSoilMoisture(numDays, rainfallTable)
    decayRate = 0.85;
    moisture = zeros(numDays,1);
    moisture(1) = 20; % starting point

    for i = 2:numDays
        moisture(i) = decayRate * moisture(i-1) + 0.6*rainfallTable.Rainfall_mm(i);
    end

    soilMoisture = table(rainfallTable.Date, round(moisture,1), ...
        'VariableNames', {'Date','SoilMoisture_pct'});
end

function catalog = loadLandslideCatalog(filepath)
    if nargin >= 1 && isfile(filepath)
        catalog = readtable(filepath);
        catalog = rmmissing(catalog);
    else
        warning('loadLandslideCatalog:fileNotFound', ...
            'CSV not found at "%s" -- using synthetic placeholder catalog.', string(filepath));
        eventDate = datetime({'2023-07-12','2023-08-03','2024-06-21'})';
        location  = {'Sector A','Sector B','Sector A'}';
        severity  = {'Moderate','Severe','Minor'}';
        catalog = table(eventDate, location, severity, 'VariableNames', {'EventDate','Location','Severity'});
    end
end

function FS = computeSlopeStability(slopeAngle_deg, cohesion_kPa, soilMoisture_pct)
    unitWeight = 18; % kN/m^3, typical soil unit weight
    soilDepth = 2;   % m, depth of potential failure plane
    phi = 30;        % degrees, internal friction angle

    beta = deg2rad(slopeAngle_deg);
    u = (soilMoisture_pct/100) .* unitWeight .* soilDepth .* 0.5; % pore pressure proxy

    numerator = cohesion_kPa + (unitWeight.*soilDepth.*cos(beta).^2 - u).*tand(phi);
    denominator = unitWeight.*soilDepth.*sin(beta).*cos(beta);

    FS = numerator ./ denominator;
    FS(denominator <= 0) = Inf;
end

function riskFactor = computeRainfallRiskFactor(rainfallTable, windowDays)
    if nargin < 2, windowDays = 3; end
    n = height(rainfallTable);
    cumulativeRain = zeros(n,1);
    for i = 1:n
        startIdx = max(1, i-windowDays+1);
        cumulativeRain(i) = sum(rainfallTable.Rainfall_mm(startIdx:i));
    end
    riskFactor = min(cumulativeRain / 150, 1); % >150mm in window = extreme (risk = 1)
end

function level = classifyRiskLevel(riskScore)
    level = strings(size(riskScore));
    level(riskScore < 30) = "Low";
    level(riskScore >= 30 & riskScore < 60) = "Moderate";
    level(riskScore >= 60 & riskScore < 80) = "High";
    level(riskScore >= 80) = "Severe";
end

function [riskScore, riskLevel] = computeLandslideRiskScore(FS, rainfallRiskFactor)
    instability = max(0, min(1, (2 - FS) / 1.5));
    combined = 0.6*instability + 0.4*rainfallRiskFactor;
    riskScore = round(combined * 100, 1);
    riskLevel = classifyRiskLevel(riskScore);
end

