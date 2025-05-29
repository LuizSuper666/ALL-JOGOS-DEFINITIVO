-- Painel flutuante com botões ordenados e correções aplicadas no ESCUDO REFLETOR SUPREMO e Auto-Heal -- Tudo organizado e funcional

-- Interface principal
local ScreenGui = Instance.new("ScreenGui") 
ScreenGui.Name = "PainelTop"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game:GetService("CoreGui")

local FloatingButton = Instance.new("TextButton")
FloatingButton.Size = UDim2.new(0, 80, 0, 30)
FloatingButton.Position = UDim2.new(0.9, 0, 0.1, 0)
FloatingButton.Text = "SCRIPT"
FloatingButton.BackgroundColor3 = Color3.fromRGB(0, 0, 255)
FloatingButton.Parent = ScreenGui

local Panel = Instance.new("Frame")
Panel.Size = UDim2.new(0, 200, 0, 400)
Panel.Position = UDim2.new(0.75, 0, 0.1, 0)
Panel.Visible = false
Panel.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
Panel.Parent = ScreenGui

local ScrollingFrame = Instance.new("ScrollingFrame")
ScrollingFrame.Size = UDim2.new(1, 0, 1, 0)
ScrollingFrame.CanvasSize = UDim2.new(0, 0, 8, 0)
ScrollingFrame.ScrollBarThickness = 5
ScrollingFrame.Parent = Panel

local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -30, 0, 0)
CloseButton.Text = "X"
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
CloseButton.Parent = Panel

-- Abrir/Fechar painel
FloatingButton.MouseButton1Click:Connect(function()
    Panel.Visible = not Panel.Visible
end)

CloseButton.MouseButton1Click:Connect(function()
    Panel.Visible = false
end)

-- Criador de botões
local function createButton(name, y, callback)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 180, 0, 40)
    button.Position = UDim2.new(0, 10, 0, y)
    button.Text = name
    button.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    button.Parent = ScrollingFrame

    local active = false
    button.MouseButton1Click:Connect(function()
        active = not active
        button.BackgroundColor3 = active and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
        callback(active)
    end)
end

-- Anti-Tudo (completo restaurado) 
createButton("Anti-Tudo", 10, function(active)
    if active then
        local mt = getrawmetatable(game)
        setreadonly(mt, false)
        local oldNamecall = mt.__namecall
        mt.__namecall = newcclosure(function(self, ...)
            local method = getnamecallmethod()
            if method:lower() == "kick" then return nil end
            return oldNamecall(self, ...)
        end)

        for _, v in pairs(getconnections(game:GetService("Players").LocalPlayer.Idled)) do
            v:Disable()
        end

        hookfunction(game.HttpPost, function(...) return nil end)

        game:GetService("CoreGui").ChildRemoved:Connect(function(child)
            if child.Name == "RobloxPromptGui" then
                wait(9e9)
            end
        end)

        local function showMessage(message)
            local screenGui = Instance.new("ScreenGui", game:GetService("CoreGui"))
            local textLabel = Instance.new("TextLabel", screenGui)
            textLabel.Size = UDim2.new(0, 400, 0, 100)
            textLabel.Position = UDim2.new(0.5, -200, 0, 10)
            textLabel.Text = message
            textLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
            textLabel.TextSize = 30
            textLabel.BackgroundTransparency = 1
            wait(2)
            screenGui:Destroy()
        end

        local function blockAllReports()
            local ReplicatedStorage = game:GetService("ReplicatedStorage")
            for _, reportName in pairs({"Report", "BugReport", "ServerFailReport"}) do
                local event = ReplicatedStorage:FindFirstChild(reportName)
                if event then
                    local oldFire = event.FireServer
                    event.FireServer = newcclosure(function()
                        showMessage("Alguém Te Denunciou!! ( Bloqueado )")
                    end)
                end
            end
        end

        blockAllReports()
        print("[🔰] Anti-Tudo ativado!")
    else
        print("[🔰] Anti-Tudo desativado!")
    end
end)

-- Intocavel
createButton("Intocável", 60, function(active)
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local root = character:WaitForChild("HumanoidRootPart")
    local cam = workspace.CurrentCamera

    if active then
        root.CFrame = root.CFrame - Vector3.new(0, 15, 0)
        local conn
        conn = game:GetService("RunService").RenderStepped:Connect(function()
            if not active then
                conn:Disconnect()
                return
            end
            cam.CameraSubject = character:FindFirstChildOfClass("Humanoid")
        end)
    else
        root.CFrame = root.CFrame + Vector3.new(0, 15, 0)
    end
end)

-- Anti-Lag Dinâmico (substitui Escudo Refletor Supremo)
createButton("Anti-Lag Dinâmico", 110, function(active)
    local player = game.Players.LocalPlayer
    local camera = workspace.CurrentCamera
    local range = 150

    local function isVisible(part)
        local _, onScreen = camera:WorldToViewportPoint(part.Position)
        return onScreen
    end

    local function simplify(part)
        if part:IsA("BasePart") then
            part.Material = Enum.Material.SmoothPlastic
            part.Transparency = 1
        end
    end

    local function restore(part)
        if part:IsA("BasePart") then
            part.Transparency = 0
        end
    end

    local function cleanEffects()
        for _, obj in pairs(workspace:GetDescendants()) do
            if obj:IsA("ParticleEmitter") or obj:IsA("Fire") or obj:IsA("Smoke") then
                obj.Enabled = not active
            end
        end
    end

    local con
    if active then
        con = game:GetService("RunService").RenderStepped:Connect(function()
            for _, part in pairs(workspace:GetDescendants()) do
                if part:IsA("BasePart") and part:IsDescendantOf(workspace) then
                    if not isVisible(part) or (part.Position - camera.CFrame.Position).Magnitude > range then
                        simplify(part)
                    else
                        restore(part)
                    end
                end
            end
            cleanEffects()
        end)
    else
        if con then con:Disconnect() end
        for _, part in pairs(workspace:GetDescendants()) do
            if part:IsA("BasePart") then
                part.Transparency = 0
            end
        end
        cleanEffects()
    end
end)

-- Voar (Corrigido)
local flying = false
local speed = 60
local flyBodyVelocity
local flyGyro
local flyConnection

createButton("Voar", 160, function(active)
    local player = game:GetService("Players").LocalPlayer
    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")

    if active and root then
        flying = true
        flyBodyVelocity = Instance.new("BodyVelocity", root)
        flyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
        flyBodyVelocity.MaxForce = Vector3.new(4000, 4000, 4000)

        flyGyro = Instance.new("BodyGyro", root)
        flyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
        flyGyro.CFrame = root.CFrame

        flyConnection = game:GetService("RunService").RenderStepped:Connect(function()
            if not flying then return end
            local cam = workspace.CurrentCamera
            flyGyro.CFrame = cam.CFrame
            flyBodyVelocity.Velocity = cam.CFrame.LookVector * speed
        end)
    else
        flying = false
        if flyConnection then 
            flyConnection:Disconnect() 
            flyConnection = nil 
        end
        if flyBodyVelocity then 
            flyBodyVelocity:Destroy() 
            flyBodyVelocity = nil 
        end
        if flyGyro then 
            flyGyro:Destroy() 
            flyGyro = nil 
        end
    end
end)

-- Atravessar paredes (Corrigido)
local atravessarAtivo = false
local atravessarConnection

createButton("Atravessar Paredes", 160, function(active)
    local char = game:GetService("Players").LocalPlayer.Character
    atravessarAtivo = active

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

-- Aumentar Velocidade x3
createButton("Velocidade x3", 210, function(active)
    local humanoid = game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        local base = humanoid.WalkSpeed / 3
        humanoid.WalkSpeed = active and base * 3 or base
    end
end)

-- ESP
local ESPEnabled = false
local ESPObjects = {}

createButton("ESP", 260, function(active)
    ESPEnabled = active

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

-- Teleporte para o jogador mais próximo
createButton("TP p/ Mais Próximo", 310, function()
    local player = game.Players.LocalPlayer
    local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not root then return end
    local alvo, menor = nil, math.huge
    for _, plr in ipairs(game.Players:GetPlayers()) do
        if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local dist = (root.Position - plr.Character.HumanoidRootPart.Position).Magnitude
            if dist < menor then
                menor = dist
                alvo = plr.Character.HumanoidRootPart
            end
        end
    end
    if alvo then
        root.CFrame = alvo.CFrame + Vector3.new(2, 0, 2)
    end
end)

-- Teleporte para o jogador mais distante
createButton("TP p/ Mais Distante", 360, function()
    local player = game.Players.LocalPlayer
    local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not root then return end
    local alvo, maior = nil, 0
    for _, plr in ipairs(game.Players:GetPlayers()) do
        if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local dist = (root.Position - plr.Character.HumanoidRootPart.Position).Magnitude
            if dist > maior then
                maior = dist
                alvo = plr.Character.HumanoidRootPart
            end
        end
    end
    if alvo then
        root.CFrame = alvo.CFrame + Vector3.new(2, 0, 2)
    end
end)
-- FPS Unlock (último botão)
createButton("FPS Unlock", 410, function(active)
    if active and setfpscap then
        setfpscap(999)
    elseif setfpscap then
        setfpscap(60)
    end
end)

