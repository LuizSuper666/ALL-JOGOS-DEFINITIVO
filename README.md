-- Criando a interface flutuante local ScreenGui = Instance.new("ScreenGui", game.Players.LocalPlayer.PlayerGui) local FloatingButton = Instance.new("TextButton", ScreenGui)

FloatingButton.Size = UDim2.new(0, 80, 0, 30) FloatingButton.Position = UDim2.new(0.9, 0, 0.1, 0) FloatingButton.Text = "SCRIPT" FloatingButton.BackgroundColor3 = Color3.fromRGB(0, 0, 255)

local Panel = Instance.new("Frame", ScreenGui) Panel.Size = UDim2.new(0, 200, 0, 200) Panel.Position = UDim2.new(0.75, 0, 0.1, 0) Panel.Visible = false Panel.BackgroundColor3 = Color3.fromRGB(50, 50, 50)

local ScrollingFrame = Instance.new("ScrollingFrame", Panel) ScrollingFrame.Size = UDim2.new(1, 0, 1.2, 0) ScrollingFrame.CanvasSize = UDim2.new(0, 0, 1.5, 0) ScrollingFrame.ScrollBarThickness = 5

local CloseButton = Instance.new("TextButton", Panel) CloseButton.Size = UDim2.new(0, 30, 0, 30) CloseButton.Position = UDim2.new(1, -30, 0, 0) CloseButton.Text = "X" CloseButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

FloatingButton.MouseButton1Click:Connect(function() Panel.Visible = not Panel.Visible end)

CloseButton.MouseButton1Click:Connect(function() Panel.Visible = false end)

local function createButton(name, position, action) local button = Instance.new("TextButton", ScrollingFrame) button.Size = UDim2.new(0, 180, 0, 40) button.Position = UDim2.new(0, 10, 0, position) button.Text = name button.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

local active = false
button.MouseButton1Click:Connect(function()
    active = not active
    button.BackgroundColor3 = active and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
    action(active)
end)

end

-- Anti-Tudo createButton("Anti-Tudo", 10, function(active) if active then local mt = getrawmetatable(game) setreadonly(mt, false) local oldNamecall = mt.__namecall mt.__namecall = newcclosure(function(self, ...) local method = getnamecallmethod() if method == "Kick" or method == "kick" then return nil end return oldNamecall(self, ...) end)

local Players = game:GetService("Players")
    for _, v in pairs(getconnections(Players.LocalPlayer.Idled)) do v:Disable() end

    local oldHttpPost = hookfunction(game.HttpPost, function(...)
        return nil
    end)

    game:GetService("CoreGui").ChildRemoved:Connect(function(child)
        if child.Name == "RobloxPromptGui" then
            wait(9e9)
        end
    end)

    print("[🔰] Anti-Tudo ativado!")
else
    print("[🔰] Anti-Tudo desativado! (Mas pode precisar reiniciar)")
end

end)

-- Imortalidade local imortalAtivo = false local imortalConnection

createButton("Imortalidade", 60, function(active) local player = game:GetService("Players").LocalPlayer local char = player.Character local humanoid = char and char:FindFirstChild("Humanoid")

imortalAtivo = active

if imortalAtivo and humanoid then
    humanoid.Health = humanoid.MaxHealth
    imortalConnection = humanoid.HealthChanged:Connect(function()
        if imortalAtivo then
            humanoid.Health = humanoid.MaxHealth
        end
    end)
else
    if imortalConnection then
        imortalConnection:Disconnect()
        imortalConnection = nil
    end
end

end)

-- ✈️ Voar local flying = false local flyConnection

createButton("Voar", 110, function(active) local player = game:GetService("Players").LocalPlayer local char = player.Character local root = char and char:FindFirstChild("HumanoidRootPart")

if active and root then
    flying = true
    root.Anchored = true
    flyConnection = game:GetService("RunService").RenderStepped:Connect(function()
        local cam = workspace.CurrentCamera
        root.CFrame = root.CFrame:Lerp(cam.CFrame, 0.3)
    end)
else
    flying = false
    if flyConnection then flyConnection:Disconnect() end
    if root then root.Anchored = false end
end

end)

-- 🚪 Atravessar paredes (Corrigido) local atravessarAtivo = false local atravessarConnection

createButton("Atravessar Paredes", 110, function(active) local char = game:GetService("Players").LocalPlayer.Character atravessarAtivo = active

if atravessarAtivo then
atravessarConnection = game:GetService("RunService").Stepped:Connect(function()
if not atravessarAtivo then return end
if char then
for _, part in pairs(char:GetChildren()) do
if part:IsA("BasePart") then
part.CanCollide = false
end
end
end
end)
else
if atravessarConnection then
atravessarConnection:Disconnect()
atravessarConnection = nil
end
if char then
for _, part in pairs(char:GetChildren()) do
if part:IsA("BasePart") then
part.CanCollide = true
end
end
end
end

end)

-- ESP local ESPEnabled = false local ESPObjects = {}

createButton("ESP", 160, function(active) ESPEnabled = active

if not ESPEnabled then
    for _, obj in pairs(ESPObjects) do
        obj:Destroy()
    end
    ESPObjects = {}
    return
end

local function createESP(player)
    if player == game.Players.LocalPlayer then return end

    local char = player.Character
    if char then
        local head = char:FindFirstChild("Head")
        if head then
            local esp = Instance.new("BillboardGui", head)
            esp.Size = UDim2.new(0, 10, 0, 10)
            esp.Adornee = head
            esp.AlwaysOnTop = true

            local dot = Instance.new("Frame", esp)
            dot.Size = UDim2.new(1, 0, 1, 0)
            dot.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
            dot.BackgroundTransparency = 0
            dot.BorderSizePixel = 0

            ESPObjects[player] = esp
        end
    end
end

for _, player in pairs(game.Players:GetPlayers()) do
    createESP(player)
end

game.Players.PlayerAdded:Connect(createESP)
game.Players.PlayerRemoving:Connect(function(player)
    if ESPObjects[player] then
        ESPObjects[player]:Destroy()
        ESPObjects[player] = nil
    end
end)

end)

-- Teleporte para o jogador mais próximo createButton("Teleporte p/ Mais Próximo", 210, function() local player = game.Players.LocalPlayer local char = player.Character local root = char and char:FindFirstChild("HumanoidRootPart")

if root then
    local closestPlayer
    local closestDistance = math.huge

    for _, otherPlayer in pairs(game.Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local otherRoot = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
            if otherRoot then
                local distance = (root.Position - otherRoot.Position).Magnitude
                if distance < closestDistance then
                    closestDistance = distance
                    closestPlayer = otherRoot
                end
            end
        end
    end

    if closestPlayer then
        root.CFrame = closestPlayer.CFrame
    end
end

end)
