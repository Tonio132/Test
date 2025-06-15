-- Interface
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)

-- Botão ESP
local ESPButton = Instance.new("TextButton")
ESPButton.Size = UDim2.new(0, 150, 0, 40)
ESPButton.Position = UDim2.new(0, 10, 0, 10)
ESPButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
ESPButton.TextColor3 = Color3.new(1, 1, 1)
ESPButton.Text = "ESP: OFF"
ESPButton.Parent = ScreenGui

-- Botão AutoFarm
local FarmButton = Instance.new("TextButton")
FarmButton.Size = UDim2.new(0, 150, 0, 40)
FarmButton.Position = UDim2.new(0, 10, 0, 60)
FarmButton.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
FarmButton.TextColor3 = Color3.new(1, 1, 1)
FarmButton.Text = "AutoFarm: OFF"
FarmButton.Parent = ScreenGui

-- Variáveis
local ESP_ENABLED = false
local FARM_ENABLED = false
local esp_objects = {}

-- ESP Função
local function createESP(player)
    if player == game.Players.LocalPlayer then return end
    if player.Team == game.Players.LocalPlayer.Team then return end

    local head = player.Character and player.Character:FindFirstChild("Head")
    if not head then return end

    local Billboard = Instance.new("BillboardGui")
    Billboard.Adornee = head
    Billboard.Name = "ESP"
    Billboard.Size = UDim2.new(0, 100, 0, 40)
    Billboard.StudsOffset = Vector3.new(0, 2, 0)
    Billboard.AlwaysOnTop = true

    local nameTag = Instance.new("TextLabel", Billboard)
    nameTag.Size = UDim2.new(1, 0, 1, 0)
    nameTag.BackgroundTransparency = 1
    nameTag.Text = player.Name
    nameTag.TextColor3 = Color3.fromRGB(255, 0, 0)
    nameTag.TextStrokeTransparency = 0.5
    nameTag.Font = Enum.Font.SourceSansBold
    nameTag.TextScaled = true

    Billboard.Parent = head
    table.insert(esp_objects, Billboard)
end

local function removeAllESP()
    for _, esp in pairs(esp_objects) do
        if esp and esp.Parent then
            esp:Destroy()
        end
    end
    esp_objects = {}
end

local function updateESP()
    removeAllESP()
    if not ESP_ENABLED then return end

    for _, player in pairs(game.Players:GetPlayers()) do
        if player.Character and player.Character:FindFirstChild("Head") then
            createESP(player)
        end
    end
end

-- AutoFarm Função simples
local function autoFarm()
    while FARM_ENABLED do
        pcall(function()
            local character = game.Players.LocalPlayer.Character
            local root = character:FindFirstChild("HumanoidRootPart")
            for _, mob in pairs(game:GetService("Workspace").Enemies:GetChildren()) do
                if mob:FindFirstChild("HumanoidRootPart") and mob:FindFirstChild("Humanoid") and mob.Humanoid.Health > 0 then
                    -- Move até o inimigo
                    root.CFrame = mob.HumanoidRootPart.CFrame * CFrame.new(0, 3, 5)
                    -- Atacar
                    for i = 1, 3 do
                        game:GetService("VirtualInputManager"):SendKeyEvent(true, "Z", false, game)
                        wait(0.2)
                        game:GetService("VirtualInputManager"):SendKeyEvent(false, "Z", false, game)
                    end
                    break
                end
            end
        end)
        wait(0.5)
    end
end

-- Conexões dos botões
ESPButton.MouseButton1Click:Connect(function()
    ESP_ENABLED = not ESP_ENABLED
    ESPButton.Text = ESP_ENABLED and "ESP: ON" or "ESP: OFF"
    updateESP()
end)

FarmButton.MouseButton1Click:Connect(function()
    FARM_ENABLED = not FARM_ENABLED
    FarmButton.Text = FARM_ENABLED and "AutoFarm: ON" or "AutoFarm: OFF"
    if FARM_ENABLED then
        spawn(autoFarm)
    end
end)

-- Atualização do ESP
game.Players.PlayerAdded:Connect(updateESP)
game.Players.PlayerRemoving:Connect(updateESP)
while true do
    if ESP_ENABLED then updateESP() end
    wait(5)
end
