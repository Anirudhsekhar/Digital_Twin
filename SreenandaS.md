%% Data Collection - Rainfall
numDays = 30;
rng(42); %makes repeatable - same "random" numbers every run
startDate = datetime('today') - days(numDays-1);
dates = (startDate:datetime('today'))';
baseRain = abs(5*randn(numDays,1));
rainfall = round(baseRain,1);
rainfallTable = table(dates, rainfall, 'VariableNames',{'Date','Rainfall_mm'});
disp(rainfallTable)
burstMask = rand(numDays,1)<0.15; %15% of days are "storm days"
burstRain = burstMask.*(40+30*rand(numDays,1));
rainfall=round(baseRain+burstRain,1);

Data Collection - Terrain
slopeAngle = 15 + 35*rand(); %one slope,15-50 degrees
cohesion = 5 + 20*rand();    %kPa
disp(table(slopeAngle,cohesion))


Data Collection - Soil Moisture
decayRate = 0.85;
moisture = zeros(numDays,1);
moisture(1) =20; %starting point

for i=2:numDays
    moisture(i) = decayRate * moisture(i-1) + 0.6*rainfallTable.Rainfall_mm(i);
end
plot(moisture)  %SEE it before trusting it

function rainfallTable = generateSyntheticRainfall(numDays,seed)
rng(seed);
startDate=datetime('today') - days(numDays-1);
dates =(startDate:datetime('today'))';
baseRain = abs(5*randn(numDays,1));
burstMask = rand(numDays,1)<0.15;
burstRain = burstMask .* (40 + 30 * rand(numDays, 1));
rainfall=round(baseRain+burstRain,1);
rainfallTable=table(dates,rainfall,'VariableNames',{'Date','Rainfall_mm'});
end
function terrain = generateSyntheticTerrain(numPoints)
slopeAngle = 15 + 35*rand(numPoints,1);   % 15-50 degrees
cohesion = 5 + 20*rand(numPoints,1);      % kPa
terrain = table(slopeAngle, cohesion, 'VariableNames', {'SlopeAngle_deg','SoilCohesion_kPa'});
end

function soilMoisture = generateSyntheticSoilMoisture(numDays, rainfallTable)
decayRate = 0.85;
moisture = zeros(numDays,1);
moisture(1) = 20;   % starting point

for i = 2:numDays
    moisture(i) = decayRate * moisture(i-1) + 0.6*rainfallTable.Rainfall_mm(i);
end

soilMoisture = table(rainfallTable.Date, round(moisture,1), ...
    'VariableNames', {'Date','SoilMoisture_pct'});
end

function FS = computeSlopeStability(slopeAngle_deg, cohesion_kPa, soilMoisture_pct)
unitWeight = 18;   % kN/m^3, typical soil unit weight
soilDepth = 2;     % m, depth of potential failure plane
phi = 30;          % degrees, internal friction angle

beta = deg2rad(slopeAngle_deg);
u = (soilMoisture_pct/100) .* unitWeight .* soilDepth .* 0.5;  % pore pressure proxy

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
riskFactor = min(cumulativeRain / 150, 1);   % >150mm in window = extreme (risk = 1)
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

