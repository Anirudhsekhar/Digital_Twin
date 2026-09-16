%% ============================================================
% LANDSLIDE SLOPE STABILITY DIGITAL TWIN
% MATLAB DASHBOARD WITH DYNAMIC EARLY-WARNING FORECAST
% ============================================================

clear;
clc;
close all;

%% ============================================================
% 1. GEOGRAPHICAL GRID
% ============================================================

gridSize = 100;

[x,y] = meshgrid(1:gridSize,1:gridSize);

%% ============================================================
% 2. TERRAIN / ELEVATION
% ============================================================

terrain = 150 - 0.8*x ...
    + 10*exp(-((x-70).^2 + (y-50).^2)/500);

%% ============================================================
% 3. TERRAIN GRADIENT
% ============================================================

[dzdx,dzdy] = gradient(terrain);

%% ============================================================
% 4. SLOPE CALCULATION
% ============================================================

slope = atan(sqrt(dzdx.^2 + dzdy.^2));

slopeDegrees = rad2deg(slope);

%% ============================================================
% 5. TERRAIN INFORMATION
% ============================================================

fprintf("\n");
fprintf("============================================\n");
fprintf("        TERRAIN INFORMATION\n");
fprintf("============================================\n");

fprintf("Grid size: %d x %d\n",gridSize,gridSize);

fprintf("Minimum elevation: %.2f m\n", ...
    min(terrain(:)));

fprintf("Maximum elevation: %.2f m\n", ...
    max(terrain(:)));

fprintf("Average elevation: %.2f m\n", ...
    mean(terrain(:)));

fprintf("\n");
fprintf("        SLOPE INFORMATION\n");

fprintf("Minimum slope: %.2f degrees\n", ...
    min(slopeDegrees(:)));

fprintf("Maximum slope: %.2f degrees\n", ...
    max(slopeDegrees(:)));

fprintf("Average slope: %.2f degrees\n", ...
    mean(slopeDegrees(:)));

%% ============================================================
% 6. 3D TERRAIN
% ============================================================

figure( ...
    'Name','3D Terrain', ...
    'NumberTitle','off');

surf(x,y,terrain);

xlabel("X");
ylabel("Y");
zlabel("Elevation (m)");

title("3D Terrain");

colorbar;

shading interp;

axis tight;

view(35.9,-0.2);

drawnow;

%% ============================================================
% 7. ELEVATION MAP
% ============================================================

figure( ...
    'Name','Elevation Map', ...
    'NumberTitle','off');

imagesc(x(1,:),y(:,1),terrain);

xlabel("X");
ylabel("Y");

title("Elevation Map");

colorbar;

axis equal;
axis tight;

drawnow;

%% ============================================================
% 8. TERRAIN CONTOUR
% ============================================================

figure( ...
    'Name','Terrain Contour', ...
    'NumberTitle','off');

contourf(x,y,terrain,20);

xlabel("X");
ylabel("Y");

title("Terrain Elevation Contours");

colorbar;

axis equal;
axis tight;

drawnow;

%% ============================================================
% 9. SLOPE MAP
% ============================================================

figure( ...
    'Name','Slope Angle Map', ...
    'NumberTitle','off');

imagesc( ...
    x(1,:), ...
    y(:,1), ...
    slopeDegrees);

xlabel("X");
ylabel("Y");

title("Slope Angle Map");

colorbar;

axis equal;
axis tight;

drawnow;

%% ============================================================
% 10. SOIL PROPERTIES
% ============================================================

cohesion = 15;          % kPa
frictionAngle = 30;     % degrees
soilDensity = 1800;     % kg/m^3
soilDepth = 5;          % meters

%% ============================================================
% 11. SOIL UNIT WEIGHT
% ============================================================

gamma = soilDensity * 9.81 / 1000;

% gamma is in kN/m^3

%% ============================================================
% 12. CONVERT ANGLES TO RADIANS
% ============================================================

slopeRadians = deg2rad(slopeDegrees);

frictionRadians = deg2rad(frictionAngle);

%% ============================================================
% 13. DRY FACTOR OF SAFETY
% ============================================================

FoS = ...
    (cohesion + ...
    gamma*soilDepth.* ...
    (cos(slopeRadians).^2).* ...
    tan(frictionRadians)) ...
    ./ ...
    (gamma*soilDepth.* ...
    sin(slopeRadians).* ...
    cos(slopeRadians));

%% ============================================================
% 14. DRY FACTOR OF SAFETY INFORMATION
% ============================================================

fprintf("\n");
fprintf("============================================\n");
fprintf("        DRY FACTOR OF SAFETY\n");
fprintf("============================================\n");

fprintf("Minimum FoS: %.2f\n", ...
    min(FoS(:)));

fprintf("Maximum FoS: %.2f\n", ...
    max(FoS(:)));

fprintf("Average FoS: %.2f\n", ...
    mean(FoS(:)));

%% ============================================================
% 15. DRY FACTOR OF SAFETY MAP
% ============================================================

figure( ...
    'Name','Dry Factor of Safety Map', ...
    'NumberTitle','off');

imagesc( ...
    x(1,:), ...
    y(:,1), ...
    FoS);

xlabel("X");
ylabel("Y");

title("Factor of Safety - Dry Condition");

colorbar;

axis equal;
axis tight;

drawnow;

%% ============================================================
% 16. RAINFALL DATA
% ============================================================

% Rainfall during each hour

rainfall = [0 10 20 30 40 60 80 100];

time = 0:7;

%% ============================================================
% 17. RAINFALL EVENT
% ============================================================

figure( ...
    'Name','Rainfall Event', ...
    'NumberTitle','off');

bar(time,rainfall);

xlabel("Time (hours)");

ylabel("Rainfall (mm)");

title("Rainfall Event");

grid on;

drawnow;

%% ============================================================
% 18. INFILTRATION
% ============================================================

infiltrationFraction = 0.60;

infiltratedRain = ...
    rainfall * infiltrationFraction;

%% ============================================================
% 19. SOIL WATER CAPACITY
% ============================================================

soilWaterCapacity = 150;

soilWater = ...
    zeros(size(rainfall));

%% ============================================================
% 20. SOIL WATER CALCULATION
% ============================================================

for t = 1:length(rainfall)

    if t == 1

        soilWater(t) = ...
            min( ...
            infiltratedRain(t), ...
            soilWaterCapacity);

    else

        soilWater(t) = ...
            min( ...
            soilWater(t-1) + ...
            infiltratedRain(t), ...
            soilWaterCapacity);

    end

end

%% ============================================================
% 21. SOIL SATURATION
% ============================================================

soilSaturation = ...
    soilWater / soilWaterCapacity;

soilSaturation = ...
    min(soilSaturation,1);

%% ============================================================
% 22. SOIL SATURATION GRAPH
% ============================================================

figure( ...
    'Name','Soil Saturation', ...
    'NumberTitle','off');

plot( ...
    time, ...
    soilSaturation, ...
    "-o", ...
    "LineWidth",2);

xlabel("Time (hours)");

ylabel("Soil Saturation");

title("Soil Saturation During Rainfall");

ylim([0 1]);

grid on;

drawnow;

%% ============================================================
% 23. PORE WATER PRESSURE
% ============================================================

gammaWater = 9.81;

porePressure = ...
    zeros(size(rainfall));

for t = 1:length(rainfall)

    porePressure(t) = ...
        soilSaturation(t) * ...
        gammaWater * ...
        soilDepth;

end

%% ============================================================
% 24. PORE PRESSURE GRAPH
% ============================================================

figure( ...
    'Name','Pore Water Pressure', ...
    'NumberTitle','off');

plot( ...
    time, ...
    porePressure, ...
    "-o", ...
    "LineWidth",2);

xlabel("Time (hours)");

ylabel("Pore Water Pressure (kPa)");

title("Pore Water Pressure During Rainfall");

grid on;

drawnow;

%% ============================================================
% 25. FACTOR OF SAFETY DURING RAINFALL
% ============================================================

FoS_time = ...
    zeros( ...
    gridSize, ...
    gridSize, ...
    length(rainfall));

%% ============================================================
% 26. SAFE SLOPE VALUE
% ============================================================

slopeRadiansSafe = ...
    max( ...
    slopeRadians, ...
    deg2rad(0.5));

%% ============================================================
% 27. WET FACTOR OF SAFETY CALCULATION
% ============================================================

for t = 1:length(rainfall)

    u = ...
        soilSaturation(t) * ...
        gammaWater * ...
        soilDepth .* ...
        cos(slopeRadiansSafe).^2;

    effectiveNormalStress = ...
        gamma * soilDepth .* ...
        cos(slopeRadiansSafe).^2 ...
        - u;

    FoS_time(:,:,t) = ...
        ( ...
        cohesion + ...
        effectiveNormalStress .* ...
        tan(frictionRadians) ...
        ) ...
        ./ ...
        ( ...
        gamma * soilDepth .* ...
        sin(slopeRadiansSafe) .* ...
        cos(slopeRadiansSafe) ...
        );

end

%% ============================================================
% 28. WET CONDITION INFORMATION
% ============================================================

finalFoS = ...
    FoS_time(:,:,end);

fprintf("\n");
fprintf("============================================\n");
fprintf("        FINAL WET CONDITION\n");
fprintf("============================================\n");

fprintf("Minimum FoS: %.2f\n", ...
    min(finalFoS(:)));

fprintf("Maximum FoS: %.2f\n", ...
    max(finalFoS(:)));

fprintf("Average FoS: %.2f\n", ...
    mean(finalFoS(:)));

%% ============================================================
% 29. FINAL WET FACTOR OF SAFETY MAP
% ============================================================

figure( ...
    'Name','Wet Factor of Safety', ...
    'NumberTitle','off');

imagesc( ...
    x(1,:), ...
    y(:,1), ...
    finalFoS);

xlabel("X");
ylabel("Y");

title("Factor of Safety After Rainfall");

colorbar;

axis equal;
axis tight;

drawnow;

%% ============================================================
% 30. MINIMUM AND AVERAGE FoS OVER TIME
% ============================================================

minimumFoS = ...
    zeros(size(rainfall));

averageFoS = ...
    zeros(size(rainfall));

for t = 1:length(rainfall)

    minimumFoS(t) = ...
        min(FoS_time(:,:,t),[],"all");

    averageFoS(t) = ...
        mean(FoS_time(:,:,t),"all");

end

%% ============================================================
% 31. FoS TREND GRAPH
% ============================================================

figure( ...
    'Name','Slope Stability Over Time', ...
    'NumberTitle','off');

plot( ...
    time, ...
    minimumFoS, ...
    "-o", ...
    "LineWidth",2);

xlabel("Time (hours)");

ylabel("Minimum Factor of Safety");

title("Slope Stability During Rainfall");

grid on;

drawnow;

%% ============================================================
% 32. SIMULATED INFRASTRUCTURE
% ============================================================

rng(1);

% 24 simulated houses

houses = ...
    [ ...
    randi([10 90],24,1), ...
    randi([10 90],24,1) ...
    ];

% Simulated road

roadX = ...
    linspace(5,95,150)';

roadY = ...
    20 + ...
    0.35*roadX + ...
    8*sin(roadX/12);

roads = ...
    [roadX roadY];

%% ============================================================
% 33. DASHBOARD
% ============================================================

dashboard = uifigure( ...
    'Name','Landslide Digital Twin Dashboard', ...
    'Position',[100 50 1250 800], ...
    'Color',[0.06 0.07 0.09]);

%% ============================================================
% 34. MAIN GRID
% ============================================================

mainGrid = uigridlayout( ...
    dashboard,[6 3]);

mainGrid.RowHeight = ...
    {65,'1x',105,175,190,60};

mainGrid.ColumnWidth = ...
    {'1x','1x','1x'};

mainGrid.Padding = [15 15 15 15];

mainGrid.RowSpacing = 10;

mainGrid.ColumnSpacing = 10;

%% ============================================================
% 35. HEADER
% ============================================================

headerPanel = uipanel(mainGrid);

headerPanel.Layout.Row = 1;

headerPanel.Layout.Column = [1 3];

headerPanel.BackgroundColor = ...
    [0.10 0.11 0.14];

headerPanel.BorderType = "none";

headerGrid = uigridlayout( ...
    headerPanel,[1 2]);

headerGrid.ColumnWidth = ...
    {'1x','1x'};

titleLabel = uilabel(headerGrid);

titleLabel.Text = ...
    "LANDSLIDE DIGITAL TWIN";

titleLabel.FontSize = 24;

titleLabel.FontWeight = "bold";

titleLabel.FontColor = ...
    [0.95 0.95 0.95];

titleLabel.VerticalAlignment = ...
    "center";

subtitleLabel = uilabel(headerGrid);

subtitleLabel.Text = ...
    "Slope Stability Monitoring & Early Warning";

subtitleLabel.HorizontalAlignment = ...
    "right";

subtitleLabel.VerticalAlignment = ...
    "center";

subtitleLabel.FontSize = 13;

subtitleLabel.FontColor = ...
    [0.65 0.67 0.72];

%% ============================================================
% 36. RISK MAP PANEL
% ============================================================

mapPanel = uipanel(mainGrid);

mapPanel.Layout.Row = 2;

mapPanel.Layout.Column = [1 2];

mapPanel.Title = "LIVE RISK MAP";

mapPanel.FontWeight = "bold";

mapPanel.BackgroundColor = ...
    [0.08 0.09 0.11];

mapPanel.ForegroundColor = ...
    [0.9 0.9 0.9];

dashboardAxes = uiaxes(mapPanel);

dashboardAxes.Position = ...
    [10 5 790 275];

dashboardAxes.XColor = ...
    [0.7 0.7 0.7];

dashboardAxes.YColor = ...
    [0.7 0.7 0.7];

dashboardAxes.Color = ...
    [0.08 0.09 0.11];

xlabel(dashboardAxes,"X");
ylabel(dashboardAxes,"Y");

title(dashboardAxes,"Current Landslide Risk");

%% ============================================================
% 37. STATUS PANEL
% ============================================================

statusPanel = uipanel(mainGrid);

statusPanel.Layout.Row = 2;

statusPanel.Layout.Column = 3;

statusPanel.Title = "CURRENT STATUS";

statusPanel.FontWeight = "bold";

statusPanel.BackgroundColor = ...
    [0.08 0.09 0.11];

statusPanel.ForegroundColor = ...
    [0.9 0.9 0.9];

statusGrid = uigridlayout( ...
    statusPanel,[5 2]);

statusGrid.RowHeight = ...
    {'1x','1x','1x','1x','1x'};

statusGrid.ColumnWidth = ...
    {'1.3x','1x'};

uilabel(statusGrid, ...
    'Text',"RAINFALL", ...
    'FontWeight',"bold", ...
    'FontColor',[0.65 0.67 0.72]);

rainfallLabel = uilabel(statusGrid);

rainfallLabel.Text = "0 mm";

rainfallLabel.FontSize = 18;

rainfallLabel.FontWeight = "bold";

rainfallLabel.FontColor = ...
    [0.9 0.9 0.9];

uilabel(statusGrid, ...
    'Text',"SATURATION", ...
    'FontWeight',"bold", ...
    'FontColor',[0.65 0.67 0.72]);

saturationLabel = uilabel(statusGrid);

saturationLabel.Text = "0 %";

saturationLabel.FontSize = 18;

saturationLabel.FontWeight = "bold";

saturationLabel.FontColor = ...
    [0.9 0.9 0.9];

uilabel(statusGrid, ...
    'Text',"PORE PRESSURE", ...
    'FontWeight',"bold", ...
    'FontColor',[0.65 0.67 0.72]);

pressureLabel = uilabel(statusGrid);

pressureLabel.Text = "0 kPa";

pressureLabel.FontSize = 18;

pressureLabel.FontWeight = "bold";

pressureLabel.FontColor = ...
    [0.9 0.9 0.9];

uilabel(statusGrid, ...
    'Text',"MINIMUM FoS", ...
    'FontWeight',"bold", ...
    'FontColor',[0.65 0.67 0.72]);

fosLabel = uilabel(statusGrid);

fosLabel.Text = "0";

fosLabel.FontSize = 18;

fosLabel.FontWeight = "bold";

fosLabel.FontColor = ...
    [0.9 0.9 0.9];

uilabel(statusGrid, ...
    'Text',"RISK LEVEL", ...
    'FontWeight',"bold", ...
    'FontColor',[0.65 0.67 0.72]);

riskLabel = uilabel(statusGrid);

riskLabel.Text = "LOW";

riskLabel.FontSize = 18;

riskLabel.FontWeight = "bold";

riskLabel.FontColor = ...
    [0.3 0.8 0.45];

%% ============================================================
% 38. RAINFALL SIMULATION CONTROL
% ============================================================

controlPanel = uipanel(mainGrid);

controlPanel.Layout.Row = 3;

controlPanel.Layout.Column = [1 2];

controlPanel.Title = ...
    "RAINFALL SIMULATION CONTROL";

controlPanel.FontWeight = "bold";

controlPanel.BackgroundColor = ...
    [0.08 0.09 0.11];

controlPanel.ForegroundColor = ...
    [0.9 0.9 0.9];

controlGrid = uigridlayout( ...
    controlPanel,[2 2]);

controlGrid.RowHeight = ...
    {'1x','1x'};

controlGrid.ColumnWidth = ...
    {'1x',180};

hourLabel = uilabel(controlGrid);

hourLabel.Text = "HOUR 0";

hourLabel.FontSize = 18;

hourLabel.FontWeight = "bold";

hourLabel.FontColor = ...
    [0.9 0.9 0.9];

hourLabel.VerticalAlignment = ...
    "center";

rainValueLabel = uilabel(controlGrid);

rainValueLabel.Text = ...
    "Rainfall this hour: 0 mm";

rainValueLabel.FontSize = 14;

rainValueLabel.FontColor = ...
    [0.65 0.67 0.72];

rainValueLabel.HorizontalAlignment = ...
    "right";

timeSlider = uislider(controlGrid);

timeSlider.Limits = ...
    [1 length(rainfall)];

timeSlider.Value = 1;

timeSlider.MajorTicks = ...
    1:length(rainfall);

timeSlider.MinorTicks = [];

timeSlider.Layout.Column = [1 2];

%% ============================================================
% 39. INFRASTRUCTURE PANEL
% ============================================================

infrastructurePanel = uipanel(mainGrid);

infrastructurePanel.Layout.Row = 3;

infrastructurePanel.Layout.Column = 3;

infrastructurePanel.Title = ...
    "INFRASTRUCTURE IMPACT";

infrastructurePanel.FontWeight = "bold";

infrastructurePanel.BackgroundColor = ...
    [0.08 0.09 0.11];

infrastructurePanel.ForegroundColor = ...
    [0.9 0.9 0.9];

infraGrid = uigridlayout( ...
    infrastructurePanel,[3 2]);

uilabel(infraGrid, ...
    'Text',"AFFECTED AREA", ...
    'FontWeight',"bold", ...
    'FontColor',[0.65 0.67 0.72]);

areaLabel = uilabel(infraGrid);

areaLabel.Text = "0 km²";

areaLabel.FontSize = 17;

areaLabel.FontWeight = "bold";

areaLabel.FontColor = ...
    [0.9 0.9 0.9];

uilabel(infraGrid, ...
    'Text',"HOUSES", ...
    'FontWeight',"bold", ...
    'FontColor',[0.65 0.67 0.72]);

housesLabel = uilabel(infraGrid);

housesLabel.Text = "0";

housesLabel.FontSize = 17;

housesLabel.FontWeight = "bold";

housesLabel.FontColor = ...
    [0.9 0.9 0.9];

uilabel(infraGrid, ...
    'Text',"ROADS", ...
    'FontWeight',"bold", ...
    'FontColor',[0.65 0.67 0.72]);

roadsLabel = uilabel(infraGrid);

roadsLabel.Text = "0";

roadsLabel.FontSize = 17;

roadsLabel.FontWeight = "bold";

roadsLabel.FontColor = ...
    [0.9 0.9 0.9];

%% ============================================================
% 40. STABILITY & RAINFALL ANALYTICS
% ============================================================

trendPanel = uipanel(mainGrid);

trendPanel.Layout.Row = 4;

trendPanel.Layout.Column = [1 3];

trendPanel.Title = ...
    "STABILITY & RAINFALL ANALYTICS";

trendPanel.FontWeight = "bold";

trendPanel.BackgroundColor = ...
    [0.08 0.09 0.11];

trendPanel.ForegroundColor = ...
    [0.9 0.9 0.9];

trendGrid = uigridlayout( ...
    trendPanel,[1 2]);

%% ============================================================
% 41. FoS TREND AXES
% ============================================================

fosAxes = uiaxes(trendGrid);

fosAxes.XColor = ...
    [0.7 0.7 0.7];

fosAxes.YColor = ...
    [0.7 0.7 0.7];

fosAxes.Color = ...
    [0.08 0.09 0.11];

xlabel(fosAxes,"Time (hours)");

ylabel(fosAxes,"Minimum FoS");

title(fosAxes,"Slope Stability");

grid(fosAxes,"on");

plot( ...
    fosAxes, ...
    time, ...
    minimumFoS, ...
    "-o", ...
    "LineWidth",2);

hold(fosAxes,"on");

currentPoint = plot( ...
    fosAxes, ...
    time(1), ...
    minimumFoS(1), ...
    "o", ...
    "MarkerSize",9, ...
    "LineWidth",2);

hold(fosAxes,"off");

%% ============================================================
% 42. RAINFALL AXES
% ============================================================

rainAxes = uiaxes(trendGrid);

rainAxes.XColor = ...
    [0.7 0.7 0.7];

rainAxes.YColor = ...
    [0.7 0.7 0.7];

rainAxes.Color = ...
    [0.08 0.09 0.11];

xlabel(rainAxes,"Time (hours)");

ylabel(rainAxes,"Rainfall (mm)");

title(rainAxes,"Rainfall Event");

grid(rainAxes,"on");

bar( ...
    rainAxes, ...
    time, ...
    rainfall);

%% ============================================================
% 43. EARLY-WARNING FORECAST PANEL
% ============================================================

forecastPanel = uipanel(mainGrid);

forecastPanel.Layout.Row = 5;

forecastPanel.Layout.Column = [1 3];

forecastPanel.Title = ...
    "EARLY-WARNING FORECAST - NEXT 4 HOURS";

forecastPanel.FontWeight = "bold";

forecastPanel.BackgroundColor = ...
    [0.08 0.09 0.11];

forecastPanel.ForegroundColor = ...
    [0.9 0.9 0.9];

forecastGrid = uigridlayout( ...
    forecastPanel,[1 2]);

forecastGrid.ColumnWidth = ...
    {'1x',300};

%% ============================================================
% 44. FORECAST AXES
% ============================================================

forecastAxes = uiaxes(forecastGrid);

forecastAxes.XColor = ...
    [0.7 0.7 0.7];

forecastAxes.YColor = ...
    [0.7 0.7 0.7];

forecastAxes.Color = ...
    [0.08 0.09 0.11];

xlabel(forecastAxes,"Time (hours)");

ylabel(forecastAxes,"Minimum FoS");

title(forecastAxes,"Observed vs Forecast Stability");

grid(forecastAxes,"on");

%% ============================================================
% 45. FORECAST INFORMATION
% ============================================================

forecastTextGrid = uigridlayout( ...
    forecastGrid,[2 1]);

forecastStatusLabel = ...
    uilabel(forecastTextGrid);

forecastStatusLabel.FontSize = 15;

forecastStatusLabel.FontWeight = "bold";

forecastStatusLabel.WordWrap = "on";

forecastStatusLabel.VerticalAlignment = "top";

forecastDetailLabel = ...
    uilabel(forecastTextGrid);

forecastDetailLabel.FontSize = 12;

forecastDetailLabel.WordWrap = "on";

forecastDetailLabel.FontColor = ...
    [0.65 0.67 0.72];

forecastDetailLabel.VerticalAlignment = "top";

%% ============================================================
% 46. RECOMMENDED ACTION PANEL
% ============================================================

actionPanel = uipanel(mainGrid);

actionPanel.Layout.Row = 6;

actionPanel.Layout.Column = [1 3];

actionPanel.BackgroundColor = ...
    [0.10 0.11 0.14];

actionPanel.BorderType = "none";

actionGrid = uigridlayout( ...
    actionPanel,[1 2]);

actionGrid.ColumnWidth = ...
    {220,'1x'};

actionTitle = uilabel(actionGrid);

actionTitle.Text = ...
    "RECOMMENDED ACTION";

actionTitle.FontSize = 15;

actionTitle.FontWeight = "bold";

actionTitle.FontColor = ...
    [0.65 0.67 0.72];

actionTitle.VerticalAlignment = ...
    "center";

actionLabel = uilabel(actionGrid);

actionLabel.Text = ...
    "MONITOR REGION";

actionLabel.FontSize = 20;

actionLabel.FontWeight = "bold";

actionLabel.FontColor = ...
    [0.3 0.8 0.45];

actionLabel.VerticalAlignment = ...
    "center";

%% ============================================================
% 47. CONNECT SLIDER TO DASHBOARD
% ============================================================

timeSlider.ValueChangedFcn = ...
    @(src,event) updateDashboard( ...
    src, ...
    dashboardAxes, ...
    fosAxes, ...
    currentPoint, ...
    rainfallLabel, ...
    rainValueLabel, ...
    saturationLabel, ...
    pressureLabel, ...
    fosLabel, ...
    riskLabel, ...
    areaLabel, ...
    housesLabel, ...
    roadsLabel, ...
    hourLabel, ...
    actionLabel, ...
    forecastAxes, ...
    forecastStatusLabel, ...
    forecastDetailLabel, ...
    FoS_time, ...
    rainfall, ...
    soilSaturation, ...
    porePressure, ...
    time, ...
    x, ...
    y, ...
    houses, ...
    roads, ...
    infiltrationFraction, ...
    soilWaterCapacity, ...
    gammaWater, ...
    soilDepth, ...
    gamma, ...
    cohesion, ...
    frictionRadians, ...
    slopeRadiansSafe);

%% ============================================================
% 48. INITIAL DASHBOARD UPDATE
% ============================================================

updateDashboard( ...
    timeSlider, ...
    dashboardAxes, ...
    fosAxes, ...
    currentPoint, ...
    rainfallLabel, ...
    rainValueLabel, ...
    saturationLabel, ...
    pressureLabel, ...
    fosLabel, ...
    riskLabel, ...
    areaLabel, ...
    housesLabel, ...
    roadsLabel, ...
    hourLabel, ...
    actionLabel, ...
    forecastAxes, ...
    forecastStatusLabel, ...
    forecastDetailLabel, ...
    FoS_time, ...
    rainfall, ...
    soilSaturation, ...
    porePressure, ...
    time, ...
    x, ...
    y, ...
    houses, ...
    roads, ...
    infiltrationFraction, ...
    soilWaterCapacity, ...
    gammaWater, ...
    soilDepth, ...
    gamma, ...
    cohesion, ...
    frictionRadians, ...
    slopeRadiansSafe);

%% ============================================================
% END OF MAIN SCRIPT
% ============================================================


%% ============================================================
% DASHBOARD UPDATE FUNCTION
% ============================================================

function updateDashboard( ...
    timeSlider, ...
    dashboardAxes, ...
    fosAxes, ...
    currentPoint, ...
    rainfallLabel, ...
    rainValueLabel, ...
    saturationLabel, ...
    pressureLabel, ...
    fosLabel, ...
    riskLabel, ...
    areaLabel, ...
    housesLabel, ...
    roadsLabel, ...
    hourLabel, ...
    actionLabel, ...
    forecastAxes, ...
    forecastStatusLabel, ...
    forecastDetailLabel, ...
    FoS_time, ...
    rainfall, ...
    soilSaturation, ...
    porePressure, ...
    time, ...
    x, ...
    y, ...
    houses, ...
    roads, ...
    infiltrationFraction, ...
    soilWaterCapacity, ...
    gammaWater, ...
    soilDepth, ...
    gamma, ...
    cohesion, ...
    frictionRadians, ...
    slopeRadiansSafe)

%% ============================================================
% 1. GET CURRENT TIME
% ============================================================

tIndex = round(timeSlider.Value);

tIndex = ...
    max(1,min(length(rainfall),tIndex));

currentHour = time(tIndex);

%% ============================================================
% 2. GET CURRENT FoS MAP
% ============================================================

currentFoS = ...
    FoS_time(:,:,tIndex);

%% ============================================================
% 3. CREATE RISK MAP
% ============================================================

riskMap = zeros(size(currentFoS));

riskMap(currentFoS > 1.5) = 1;

riskMap(currentFoS >= 1.2 & ...
    currentFoS <= 1.5) = 2;

riskMap(currentFoS >= 1.0 & ...
    currentFoS < 1.2) = 3;

riskMap(currentFoS < 1.0) = 4;

%% ============================================================
% 4. RISK PERCENTAGES
% ============================================================

totalCells = numel(currentFoS);

extremePercentage = ...
    sum(currentFoS(:) < 1.0) ...
    / totalCells * 100;

highPercentage = ...
    sum(currentFoS(:) >= 1.0 & ...
    currentFoS(:) < 1.2) ...
    / totalCells * 100;

moderatePercentage = ...
    sum(currentFoS(:) >= 1.2 & ...
    currentFoS(:) <= 1.5) ...
    / totalCells * 100;

%% ============================================================
% 5. OVERALL RISK
% ============================================================

if extremePercentage >= 20

    overallRisk = "EXTREME";

    action = "IMMEDIATE EVACUATION";

    riskColor = [0.90 0.20 0.25];

elseif extremePercentage >= 5 || ...
        highPercentage >= 25

    overallRisk = "HIGH";

    action = "EVACUATE HIGH-RISK REGION";

    riskColor = [0.95 0.50 0.20];

elseif highPercentage >= 10 || ...
        moderatePercentage >= 25

    overallRisk = "MODERATE";

    action = "ISSUE WARNING";

    riskColor = [0.95 0.78 0.20];

else

    overallRisk = "LOW";

    action = "MONITOR REGION";

    riskColor = [0.30 0.80 0.45];

end

%% ============================================================
% 6. MINIMUM FoS
% ============================================================

currentMinimumFoS = ...
    min(currentFoS(:));

%% ============================================================
% 7. UPDATE RISK MAP
% ============================================================

cla(dashboardAxes);

imagesc( ...
    dashboardAxes, ...
    x(1,:), ...
    y(:,1), ...
    riskMap);

axis(dashboardAxes,"equal");
axis(dashboardAxes,"tight");

caxis(dashboardAxes,[1 4]);

colormap(dashboardAxes, ...
    [ ...
    0.25 0.70 0.42
    0.90 0.78 0.20
    0.92 0.48 0.18
    0.82 0.18 0.20
    ]);

title( ...
    dashboardAxes, ...
    sprintf( ...
    "Landslide Risk Map - Hour %.0f", ...
    currentHour));

%% ============================================================
% 8. AFFECTED AREA
% ============================================================

affectedCells = ...
    currentFoS < 1.2;

cellArea = 0.001;

affectedArea = ...
    sum(affectedCells(:)) * cellArea;

%% ============================================================
% 9. AFFECTED HOUSES
% ============================================================

affectedHouses = 0;

for i = 1:size(houses,1)

    hx = round(houses(i,1));
    hy = round(houses(i,2));

    hx = ...
        max(1,min(size(currentFoS,2),hx));

    hy = ...
        max(1,min(size(currentFoS,1),hy));

    houseFoS = ...
        currentFoS(hy,hx);

    if houseFoS < 1.2

        affectedHouses = ...
            affectedHouses + 1;

    end

end

%% ============================================================
% 10. AFFECTED ROAD
% ============================================================

affectedRoadPoints = 0;

for i = 1:size(roads,1)

    rx = round(roads(i,1));
    ry = round(roads(i,2));

    rx = ...
        max(1,min(size(currentFoS,2),rx));

    ry = ...
        max(1,min(size(currentFoS,1),ry));

    roadFoS = ...
        currentFoS(ry,rx);

    if roadFoS < 1.2

        affectedRoadPoints = ...
            affectedRoadPoints + 1;

    end

end

roadPercentage = ...
    affectedRoadPoints / size(roads,1) * 100;

if roadPercentage >= 20

    affectedRoads = 1;

else

    affectedRoads = 0;

end

%% ============================================================
% 11. CURRENT RAINFALL INFORMATION
% ============================================================

cumulativeRainfall = ...
    sum(rainfall(1:tIndex));

currentRainfall = ...
    rainfall(tIndex);

currentSaturation = ...
    soilSaturation(tIndex);

currentPressure = ...
    porePressure(tIndex);

%% ============================================================
% 12. UPDATE DASHBOARD TEXT
% ============================================================

rainfallLabel.Text = ...
    sprintf("%.0f mm",cumulativeRainfall);

rainValueLabel.Text = ...
    sprintf( ...
    "Rainfall this hour: %.0f mm", ...
    currentRainfall);

saturationLabel.Text = ...
    sprintf( ...
    "%.0f %%", ...
    currentSaturation * 100);

pressureLabel.Text = ...
    sprintf( ...
    "%.1f kPa", ...
    currentPressure);

fosLabel.Text = ...
    sprintf( ...
    "%.2f", ...
    currentMinimumFoS);

riskLabel.Text = ...
    overallRisk;

riskLabel.FontColor = ...
    riskColor;

areaLabel.Text = ...
    sprintf( ...
    "%.2f km²", ...
    affectedArea);

housesLabel.Text = ...
    sprintf( ...
    "%d", ...
    affectedHouses);

roadsLabel.Text = ...
    sprintf( ...
    "%d", ...
    affectedRoads);

hourLabel.Text = ...
    sprintf( ...
    "HOUR %.0f", ...
    currentHour);

actionLabel.Text = ...
    action;

actionLabel.FontColor = ...
    riskColor;

%% ============================================================
% 13. UPDATE FoS GRAPH MARKER
% ============================================================

currentPoint.XData = ...
    currentHour;

currentPoint.YData = ...
    currentMinimumFoS;

%% ============================================================
% 14. DYNAMIC EARLY-WARNING FORECAST
% ============================================================

forecastHorizon = 4;

[ ...
    forecastTime, ...
    forecastRainfall, ...
    forecastSaturation, ...
    forecastPorePressure, ...
    forecastMinimumFoS, ...
    earlyWarningActive, ...
    hoursUntilWarning, ...
    earlyWarningText] = ...
    calculateForecast( ...
    tIndex, ...
    time, ...
    rainfall, ...
    soilSaturation, ...
    infiltrationFraction, ...
    soilWaterCapacity, ...
    gammaWater, ...
    soilDepth, ...
    gamma, ...
    cohesion, ...
    frictionRadians, ...
    slopeRadiansSafe, ...
    forecastHorizon);

%% ============================================================
% 15. UPDATE FORECAST GRAPH
% ============================================================

cla(forecastAxes);

observedMinimumFoS = zeros(1,tIndex);

for j = 1:tIndex
    observedMinimumFoS(j) = min(FoS_time(:,:,j),[],"all");
end

plot( ...
    forecastAxes, ...
    time(1:tIndex), ...
    observedMinimumFoS, ...
    "-o", ...
    "LineWidth",2);

hold(forecastAxes,"on");

plot( ...
    forecastAxes, ...
    forecastTime, ...
    forecastMinimumFoS, ...
    "--o", ...
    "LineWidth",2);

yline( ...
    forecastAxes, ...
    1.2, ...
    "--", ...
    "Warning 1.2", ...
    "LabelHorizontalAlignment","left");

yline( ...
    forecastAxes, ...
    1.0, ...
    "--", ...
    "Failure 1.0", ...
    "LabelHorizontalAlignment","left");

hold(forecastAxes,"off");

legend( ...
    forecastAxes, ...
    "Observed", ...
    "Forecast", ...
    "Location","best");

xlabel(forecastAxes,"Time (hours)");

ylabel(forecastAxes,"Minimum FoS");

title( ...
    forecastAxes, ...
    sprintf( ...
    "Observed vs Forecast - From Hour %.0f", ...
    currentHour));

grid(forecastAxes,"on");

xlim( ...
    forecastAxes, ...
    [time(1) forecastTime(end)]);

%% ============================================================
% 16. UPDATE FORECAST STATUS
% ============================================================

if earlyWarningActive

    forecastStatusLabel.Text = ...
        sprintf( ...
        "EARLY WARNING: HIGH RISK IN %d HOUR(S)", ...
        hoursUntilWarning);

    forecastStatusLabel.FontColor = ...
        [0.95 0.50 0.20];

else

    forecastStatusLabel.Text = ...
        "NO CRITICAL RISK PREDICTED";

    forecastStatusLabel.FontColor = ...
        [0.30 0.80 0.45];

end

%% ============================================================
% 17. FORECAST DETAILS
% ============================================================

forecastDetailLabel.Text = sprintf( ...
    "Current hour: %.0f\n\n" + ...
    "Next 4-hour rainfall:\n" + ...
    "H%.0f  %.0f mm   |   H%.0f  %.0f mm\n" + ...
    "H%.0f  %.0f mm   |   H%.0f  %.0f mm\n\n" + ...
    "Predicted minimum FoS:\n" + ...
    "H%.0f  %.2f   |   H%.0f  %.2f\n" + ...
    "H%.0f  %.2f   |   H%.0f  %.2f\n\n" + ...
    "Predicted saturation at H%.0f: %.0f %%\n" + ...
    "Predicted pore pressure at H%.0f: %.1f kPa\n\n" + ...
    "%s", ...
    currentHour, ...
    forecastTime(1),forecastRainfall(1), ...
    forecastTime(2),forecastRainfall(2), ...
    forecastTime(3),forecastRainfall(3), ...
    forecastTime(4),forecastRainfall(4), ...
    forecastTime(1),forecastMinimumFoS(1), ...
    forecastTime(2),forecastMinimumFoS(2), ...
    forecastTime(3),forecastMinimumFoS(3), ...
    forecastTime(4),forecastMinimumFoS(4), ...
    forecastTime(4),forecastSaturation(4)*100, ...
    forecastTime(4),forecastPorePressure(4), ...
    earlyWarningText);

%% ============================================================
% 18. FORCE REFRESH
% ============================================================

drawnow;

end


%% ============================================================
% DYNAMIC FORECAST CALCULATION
% ============================================================

function [ ...
    forecastTime, ...
    forecastRainfall, ...
    forecastSaturation, ...
    forecastPorePressure, ...
    forecastMinimumFoS, ...
    earlyWarningActive, ...
    hoursUntilWarning, ...
    earlyWarningText] = ...
    calculateForecast( ...
    tIndex, ...
    time, ...
    rainfall, ...
    soilSaturation, ...
    infiltrationFraction, ...
    soilWaterCapacity, ...
    gammaWater, ...
    soilDepth, ...
    gamma, ...
    cohesion, ...
    frictionRadians, ...
    slopeRadiansSafe, ...
    forecastHorizon)

%% ============================================================
% 1. CURRENT TIME
% ============================================================

currentHour = time(tIndex);

forecastTime = ...
    currentHour + (1:forecastHorizon);

%% ============================================================
% 2. GET RAINFALL HISTORY UP TO CURRENT HOUR
% ============================================================

observedRainfall = ...
    rainfall(1:tIndex);

%% ============================================================
% 3. CALCULATE RECENT RAINFALL TREND
% ============================================================

if length(observedRainfall) >= 3

    recentRainfall = ...
        observedRainfall(end-2:end);

    rainfallTrend = ...
        mean(diff(recentRainfall));

elseif length(observedRainfall) == 2

    rainfallTrend = ...
        observedRainfall(end) - ...
        observedRainfall(end-1);

else

    rainfallTrend = 0;

end

%% ============================================================
% 4. FORECAST RAINFALL
% ============================================================

forecastRainfall = ...
    zeros(1,forecastHorizon);

lastRain = ...
    rainfall(tIndex);

for k = 1:forecastHorizon

    nextRain = ...
        lastRain + rainfallTrend;

    nextRain = ...
        max(nextRain,0);

    forecastRainfall(k) = ...
        nextRain;

    lastRain = ...
        nextRain;

end

%% ============================================================
% 5. FORECAST SOIL WATER
% ============================================================

forecastInfiltratedRain = ...
    forecastRainfall * infiltrationFraction;

forecastSoilWater = ...
    zeros(1,forecastHorizon);

lastSoilWater = ...
    soilSaturation(tIndex) * soilWaterCapacity;

for k = 1:forecastHorizon

    lastSoilWater = ...
        min( ...
        lastSoilWater + ...
        forecastInfiltratedRain(k), ...
        soilWaterCapacity);

    forecastSoilWater(k) = ...
        lastSoilWater;

end

%% ============================================================
% 6. FORECAST SATURATION
% ============================================================

forecastSaturation = ...
    forecastSoilWater / soilWaterCapacity;

forecastSaturation = ...
    min(forecastSaturation,1);

%% ============================================================
% 7. FORECAST PORE PRESSURE
% ============================================================

forecastPorePressure = ...
    forecastSaturation * ...
    gammaWater * ...
    soilDepth;

%% ============================================================
% 8. FORECAST SPATIAL FoS
% ============================================================

forecastMinimumFoS = ...
    zeros(1,forecastHorizon);

for k = 1:forecastHorizon

    u = ...
        forecastSaturation(k) * ...
        gammaWater * ...
        soilDepth .* ...
        cos(slopeRadiansSafe).^2;

    effectiveNormalStress = ...
        gamma * soilDepth .* ...
        cos(slopeRadiansSafe).^2 ...
        - u;

    forecastFoS = ...
        ( ...
        cohesion + ...
        effectiveNormalStress .* ...
        tan(frictionRadians) ...
        ) ...
        ./ ...
        ( ...
        gamma * soilDepth .* ...
        sin(slopeRadiansSafe) .* ...
        cos(slopeRadiansSafe) ...
        );

    forecastMinimumFoS(k) = ...
        min(forecastFoS(:));

end

%% ============================================================
% 9. EARLY-WARNING LOGIC
% ============================================================

warningHourIndex = ...
    find(forecastMinimumFoS < 1.2,1,"first");

failureHourIndex = ...
    find(forecastMinimumFoS < 1.0,1,"first");

if ~isempty(failureHourIndex)

    hoursUntilWarning = ...
        failureHourIndex;

    earlyWarningActive = true;

    earlyWarningText = sprintf( ...
        "CRITICAL: FoS below 1.0 predicted in %d hour(s).", ...
        hoursUntilWarning);

elseif ~isempty(warningHourIndex)

    hoursUntilWarning = ...
        warningHourIndex;

    earlyWarningActive = true;

    earlyWarningText = sprintf( ...
        "WARNING: FoS below 1.2 predicted in %d hour(s).", ...
        hoursUntilWarning);

else

    hoursUntilWarning = NaN;

    earlyWarningActive = false;

    earlyWarningText = ...
        "No critical risk predicted in the next 4 hours.";

end

end

