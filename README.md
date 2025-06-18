-- ESP Script para Roblox
-- Permite ver jogadores através das paredes com informações úteis

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Configurações do ESP
local ESPSettings = {
    Enabled = true,
    ShowBox = true,
    ShowName = true,
    ShowDistance = true,
    ShowHealth = true,
    ShowTeam = true,
    BoxColor = Color3.fromRGB(255, 0, 0),
    NameColor = Color3.fromRGB(255, 255, 255),
    TeamColor = true, -- Usar cores do time
    MaxDistance = 1000, -- Distância máxima em studs
    BoxThickness = 2,
    NameSize = 16,
    DistanceSize = 14,
    HealthSize = 14
}

-- Tabela para armazenar objetos ESP
local ESPObjects = {}

-- Função para criar GUI
local function createESPGui()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "ESP_GUI"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    
    return ScreenGui
end

-- Função para criar elementos ESP para um jogador
local function createESP(player)
    if player == LocalPlayer then return end
    
    local espFolder = Instance.new("Folder")
    espFolder.Name = "ESP_" .. player.Name
    espFolder.Parent = ESPObjects.ScreenGui
    
    -- Caixa do ESP
    local box = Instance.new("Frame")
    box.Name = "Box"
    box.BackgroundTransparency = 1
    box.BorderSizePixel = ESPSettings.BoxThickness
    box.BorderColor3 = ESPSettings.BoxColor
    box.Parent = espFolder
    
    -- Nome do jogador
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Name = "NameLabel"
    nameLabel.BackgroundTransparency = 1
    nameLabel.Font = Enum.Font.SourceSansBold
    nameLabel.TextSize = ESPSettings.NameSize
    nameLabel.TextColor3 = ESPSettings.NameColor
    nameLabel.TextStrokeTransparency = 0
    nameLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    nameLabel.Text = player.Name
    nameLabel.Parent = espFolder
    
    -- Distância
    local distanceLabel = Instance.new("TextLabel")
    distanceLabel.Name = "DistanceLabel"
    distanceLabel.BackgroundTransparency = 1
    distanceLabel.Font = Enum.Font.SourceSans
    distanceLabel.TextSize = ESPSettings.DistanceSize
    distanceLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    distanceLabel.TextStrokeTransparency = 0
    distanceLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    distanceLabel.Parent = espFolder
    
    -- Vida
    local healthLabel = Instance.new("TextLabel")
    healthLabel.Name = "HealthLabel"
    healthLabel.BackgroundTransparency = 1
    healthLabel.Font = Enum.Font.SourceSans
    healthLabel.TextSize = ESPSettings.HealthSize
    healthLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
    healthLabel.TextStrokeTransparency = 0
    healthLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    healthLabel.Parent = espFolder
    
    -- Barra de vida
    local healthBar = Instance.new("Frame")
    healthBar.Name = "HealthBar"
    healthBar.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    healthBar.BorderSizePixel = 1
    healthBar.BorderColor3 = Color3.fromRGB(0, 0, 0)
    healthBar.Parent = espFolder
    
    local healthFill = Instance.new("Frame")
    healthFill.Name = "HealthFill"
    healthFill.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    healthFill.BorderSizePixel = 0
    healthFill.Parent = healthBar
    
    ESPObjects[player] = espFolder
end

-- Função para remover ESP de um jogador
local function removeESP(player)
    if ESPObjects[player] then
        ESPObjects[player]:Destroy()
        ESPObjects[player] = nil
    end
end

-- Função para atualizar ESP
local function updateESP()
    if not ESPSettings.Enabled then return end
    
    for player, espFolder in pairs(ESPObjects) do
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") then
            local character = player.Character
            local humanoidRootPart = character.HumanoidRootPart
            local humanoid = character.Humanoid
            
            -- Calcular distância
            local distance = (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")) 
                and (LocalPlayer.Character.HumanoidRootPart.Position - humanoidRootPart.Position).Magnitude or math.huge
            
            -- Verificar se está dentro da distância máxima
            if distance > ESPSettings.MaxDistance then
                espFolder.Visible = false
                continue
            end
            
            -- Converter posição 3D para 2D
            local screenPosition, onScreen = Camera:WorldToViewportPoint(humanoidRootPart.Position)
            
            if onScreen then
                espFolder.Visible = true
                
                -- Calcular tamanho da caixa baseado na distância
                local scaleFactor = 1000 / distance
                local boxSize = Vector2.new(
                    math.clamp(scaleFactor * 4, 4, 100),
                    math.clamp(scaleFactor * 6, 6, 150)
                )
                
                -- Atualizar caixa
                if ESPSettings.ShowBox then
                    local box = espFolder:FindFirstChild("Box")
                    if box then
                        box.Visible = true
                        box.Position = UDim2.new(0, screenPosition.X - boxSize.X/2, 0, screenPosition.Y - boxSize.Y/2)
                        box.Size = UDim2.new(0, boxSize.X, 0, boxSize.Y)
                        
                        -- Cor baseada no time se habilitado
                        if ESPSettings.TeamColor and player.Team then
                            box.BorderColor3 = player.Team.TeamColor.Color
                        else
                            box.BorderColor3 = ESPSettings.BoxColor
                        end
                    end
                else
                    local box = espFolder:FindFirstChild("Box")
                    if box then box.Visible = false end
                end
                
                -- Atualizar nome
                if ESPSettings.ShowName then
                    local nameLabel = espFolder:FindFirstChild("NameLabel")
                    if nameLabel then
                        nameLabel.Visible = true
                        nameLabel.Position = UDim2.new(0, screenPosition.X, 0, screenPosition.Y - boxSize.Y/2 - 30)
                        nameLabel.Size = UDim2.new(0, 200, 0, 20)
                        nameLabel.Text = player.Name
                        
                        if ESPSettings.TeamColor and player.Team then
                            nameLabel.TextColor3 = player.Team.TeamColor.Color
                        else
                            nameLabel.TextColor3 = ESPSettings.NameColor
                        end
                    end
                else
                    local nameLabel = espFolder:FindFirstChild("NameLabel")
                    if nameLabel then nameLabel.Visible = false end
                end
                
                -- Atualizar distância
                if ESPSettings.ShowDistance then
                    local distanceLabel = espFolder:FindFirstChild("DistanceLabel")
                    if distanceLabel then
                        distanceLabel.Visible = true
                        distanceLabel.Position = UDim2.new(0, screenPosition.X, 0, screenPosition.Y + boxSize.Y/2 + 5)
                        distanceLabel.Size = UDim2.new(0, 200, 0, 20)
                        distanceLabel.Text = math.floor(distance) .. " studs"
                    end
                else
                    local distanceLabel = espFolder:FindFirstChild("DistanceLabel")
                    if distanceLabel then distanceLabel.Visible = false end
                end
                
                -- Atualizar vida
                if ESPSettings.ShowHealth then
                    local healthLabel = espFolder:FindFirstChild("HealthLabel")
                    local healthBar = espFolder:FindFirstChild("HealthBar")
                    local healthFill = healthBar and healthBar:FindFirstChild("HealthFill")
                    
                    if healthLabel and healthBar and healthFill then
                        local healthPercent = humanoid.Health / humanoid.MaxHealth
                        
                        healthLabel.Visible = true
                        healthLabel.Position = UDim2.new(0, screenPosition.X, 0, screenPosition.Y + boxSize.Y/2 + 25)
                        healthLabel.Size = UDim2.new(0, 200, 0, 20)
                        healthLabel.Text = math.floor(humanoid.Health) .. "/" .. math.floor(humanoid.MaxHealth)
                        
                        -- Cor da vida baseada na porcentagem
                        if healthPercent > 0.6 then
                            healthLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
                        elseif healthPercent > 0.3 then
                            healthLabel.TextColor3 = Color3.fromRGB(255, 255, 0)
                        else
                            healthLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
                        end
                        
                        -- Barra de vida
                        healthBar.Visible = true
                        healthBar.Position = UDim2.new(0, screenPosition.X - 25, 0, screenPosition.Y + boxSize.Y/2 + 45)
                        healthBar.Size = UDim2.new(0, 50, 0, 4)
                        
                        healthFill.Size = UDim2.new(healthPercent, 0, 1, 0)
                        healthFill.BackgroundColor3 = healthLabel.TextColor3
                    end
                else
                    local healthLabel = espFolder:FindFirstChild("HealthLabel")
                    local healthBar = espFolder:FindFirstChild("HealthBar")
                    if healthLabel then healthLabel.Visible = false end
                    if healthBar then healthBar.Visible = false end
                end
                
            else
                espFolder.Visible = false
            end
        else
            espFolder.Visible = false
        end
    end
end

-- Função para toggle ESP
local function toggleESP()
    ESPSettings.Enabled = not ESPSettings.Enabled
    
    if not ESPSettings.Enabled then
        for _, espFolder in pairs(ESPObjects) do
            espFolder.Visible = false
        end
    end
    
    print("ESP " .. (ESPSettings.Enabled and "Ativado" or "Desativado"))
end

-- Inicializar ESP
local function initializeESP()
    ESPObjects.ScreenGui = createESPGui()
    
    -- Criar ESP para jogadores existentes
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            createESP(player)
        end
    end
    
    -- Eventos para novos jogadores
    Players.PlayerAdded:Connect(createESP)
    Players.PlayerRemoving:Connect(removeESP)
    
    -- Loop de atualização
    RunService.Heartbeat:Connect(updateESP)
    
    print("ESP Inicializado!")
    print("Pressione 'P' para toggle ESP")
end

-- Controles de teclado
local UserInputService = game:GetService("UserInputService")
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.P then
        toggleESP()
    end
end)

-- Menu de configurações (opcional)
local function createSettingsMenu()
    -- Implementar GUI de configurações aqui se necessário
    -- Por enquanto, as configurações podem ser alteradas no topo do script
end

-- Inicializar
initializeESP()

-- Função para limpar ESP (útil para recarregar o script)
local function cleanupESP()
    if ESPObjects.ScreenGui then
        ESPObjects.ScreenGui:Destroy()
    end
    ESPObjects = {}
end

-- Comandos de chat (opcional)
LocalPlayer.Chatted:Connect(function(message)
    local args = string.split(message:lower(), " ")
    
    if args[1] == "/esp" then
        if args[2] == "toggle" then
            toggleESP()
        elseif args[2] == "distance" and args[3] then
            local newDistance = tonumber(args[3])
            if newDistance then
                ESPSettings.MaxDistance = newDistance
                print("Distância máxima alterada para: " .. newDistance)
            end
        elseif args[2] == "cleanup" then
            cleanupESP()
            print("ESP limpo!")
        end
    end
end)
