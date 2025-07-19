local Lighting = game:GetService("Lighting")
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local playerGui = player:WaitForChild("PlayerGui")

-- GUI de Clima
local weatherGui = Instance.new("ScreenGui", playerGui)
weatherGui.Name = "WeatherGui"

local container = Instance.new("Frame", weatherGui)
container.Size = UDim2.new(0, 110, 0, 35)
container.Position = UDim2.new(0.5, 0, 0, -10)  -- POSIÇÃO AJUSTADA PARA CIMA
container.AnchorPoint = Vector2.new(0.5, 0)
container.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
container.BackgroundTransparency = 0.2
container.BorderColor3 = Color3.new(1, 1, 1)
container.BorderSizePixel = 2

Instance.new("UICorner", container).CornerRadius = UDim.new(0, 10)

local weatherLabel = Instance.new("TextLabel", container)
weatherLabel.Size = UDim2.new(1, 0, 1, 0)
weatherLabel.BackgroundTransparency = 1
weatherLabel.Text = "☀️  30°C"
weatherLabel.Font = Enum.Font.SourceSansBold
weatherLabel.TextSize = 22
weatherLabel.TextColor3 = Color3.fromRGB(255, 255, 255)

local uiScale = Instance.new("UIScale", container)

local function ajustarUIScale()
    local size = camera.ViewportSize
    local menor = math.min(size.X, size.Y)
    uiScale.Scale = math.clamp(menor / 900, 0.75, 1.15)
end

ajustarUIScale()
camera:GetPropertyChangedSignal("ViewportSize"):Connect(ajustarUIScale)

-- Sons
local function criarSom(id, volume, looped)
    local s = Instance.new("Sound", workspace)
    s.SoundId = "rbxassetid://" .. id
    s.Volume = volume
    s.Looped = looped or false
    return s
end

local rainSound = criarSom("86400900584545", 0.1, true)
local thunderSound = criarSom("135831804432479", 0.4)
local birdsSound = criarSom("178353513", 0.2, true)
birdsSound:Play()

-- Gota de chuva base
local baseDrop = Instance.new("Part")
baseDrop.Anchored = true
baseDrop.CanCollide = false
baseDrop.Transparency = 0.6
baseDrop.BrickColor = BrickColor.new("Pastel light blue")
baseDrop.Size = Vector3.new(0.2, 1.2, 0.2)
baseDrop.TopSurface = "Smooth"
baseDrop.BottomSurface = "Smooth"
Instance.new("BlockMesh", baseDrop).Scale = Vector3.new(0.4, 2, 0.4)

-- Variáveis
local raining = false
local temperature = 30

-- Controle de duração da chuva
local rainStartTime = 0
local rainDuration = 0  -- em segundos
local minRainTime = 300 -- 5 minutos
local maxRainTime = 420 -- 7 minutos

-- Sky nublado azul
local cloudySky = Instance.new("Sky")
cloudySky.Name = "CloudySky"
cloudySky.SkyboxBk = "rbxassetid://242580474"
cloudySky.SkyboxDn = "rbxassetid://242580591"
cloudySky.SkyboxFt = "rbxassetid://242580626"
cloudySky.SkyboxLf = "rbxassetid://242580673"
cloudySky.SkyboxRt = "rbxassetid://242580713"
cloudySky.SkyboxUp = "rbxassetid://242580760"

-- Luz
local function updateLighting()
    if raining then
        Lighting.Brightness = 1
        Lighting.ClockTime = 12
        Lighting.OutdoorAmbient = Color3.fromRGB(90, 90, 90)
        Lighting.FogColor = Color3.fromRGB(200, 200, 200)  -- neblina leve e suave
        Lighting.FogStart = 50
        Lighting.FogEnd = 250
        cloudySky:Clone().Parent = Lighting
    else
        Lighting.Brightness = 2
        Lighting.ClockTime = 14
        Lighting.OutdoorAmbient = Color3.fromRGB(128, 128, 128)
        Lighting.FogStart = 0
        Lighting.FogEnd = 100000 -- sem neblina
        Lighting.FogColor = Color3.fromRGB(255, 255, 255)
        for _, obj in ipairs(Lighting:GetChildren()) do
            if obj:IsA("Sky") and obj.Name == "CloudySky" then
                obj:Destroy()
            end
        end
    end
end

-- Chuva visual
task.spawn(function()
    while true do
        if raining then
            local dropAmount = math.clamp(10 + (28 - temperature) * 2, 10, 100)
            for i = 1, dropAmount do
                task.wait(0.03)
                local drop = baseDrop:Clone()
                drop.Parent = camera
                drop.CFrame = camera.CFrame * CFrame.new(math.random(-25,25), math.random(40,55), math.random(-25,25))
                drop.Anchored = false
                drop.Touched:Connect(function() drop:Destroy() end)
                end
                else
                    task.wait(0.5)
                end
            end
        end)

-- Clima automático com limite de duração da chuva
task.spawn(function()
    while true do
        task.wait(3)
        
        local chance = math.random()
        if chance < 0.4 then
            temperature += math.random(0, 1)  -- 40% chance de sol (aumenta temperatura)
        elseif chance < 0.9 then
            temperature -= math.random(0, 1)  -- 50% chance de chuva (diminui temperatura)
        end
        temperature = math.clamp(temperature, 10, 40)
        
        if temperature <= 28 and not raining then
            raining = true
            rainStartTime = os.time()
            rainDuration = math.random(minRainTime, maxRainTime)
            updateLighting()
            rainSound:Play()
            birdsSound:Stop()
        elseif raining then
            local elapsed = os.time() - rainStartTime
            
            -- Nos últimos 1 minuto da chuva, aumente a temperatura gradualmente para terminar
            if elapsed >= rainDuration - 60 then
                temperature = math.min(temperature + 0.1, 40)
            end
            
            if elapsed >= rainDuration or temperature >= 30 then
                raining = false
                updateLighting()
                rainSound:Stop()
                birdsSound:Play()
            end
        end
        
        if raining then
            rainSound.Volume = math.clamp(0.1 + (28 - temperature) * 0.03, 0.1, 1)
        end
        
        local icon = "☀️"
        if raining then
            icon = "🌧️"
        elseif temperature == 29 then
            icon = "🌥️"
        end
        weatherLabel.Text = icon .. "  " .. math.floor(temperature) .. "°C"
    end
end)

-- Trovão aleatório
task.spawn(function()
    while true do
        task.wait(5)
        if raining and math.random() < 0.3 then
            thunderSound:Play()
        end
    end
end)
