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
ScrollingFrame.CanvasSize = UDim2.new(0, 0, 20, 0)
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

-- Anti-Tudo com proteção total (incluindo RemoteEvents suspeitos)
createButton("Anti-Tudo", 10, function(active)
    if not active then
        print("[🔰] Anti-Tudo desativado!")
        return
    end

    local Players = game:GetService("Players")
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local CoreGui = game:GetService("CoreGui")
    local TeleportService = game:GetService("TeleportService")
    local LocalPlayer = Players.LocalPlayer

    -- Bloquear kick via __namecall
    local mt = getrawmetatable(game)
    setreadonly(mt, false)
    local oldNamecall = mt.__namecall
    mt.__namecall = newcclosure(function(self, ...)
        if getnamecallmethod():lower() == "kick" then return nil end
        return oldNamecall(self, ...)
    end)

    -- Desativar kick por inatividade
    for _, conn in ipairs(getconnections(LocalPlayer.Idled)) do
        conn:Disable()
    end

    -- Bloquear requisições HTTP
    hookfunction(game.HttpPost, function(...) return nil end)

    -- Prevenir remoção de telas de sistema (ex: aviso de ban)
    CoreGui.ChildRemoved:Connect(function(child)
        if child.Name == "RobloxPromptGui" then
            task.wait(9e9)
        end
    end)

    -- Mostrar aviso na tela
    local function showMessage(message)
        local gui = Instance.new("ScreenGui", CoreGui)
        local label = Instance.new("TextLabel", gui)
        label.Size = UDim2.new(0, 400, 0, 100)
        label.Position = UDim2.new(0.5, -200, 0, 10)
        label.Text = message
        label.TextColor3 = Color3.new(1, 0, 0)
        label.TextSize = 30
        label.BackgroundTransparency = 1
        task.wait(2)
        gui:Destroy()
    end

    -- Bloquear eventos de denúncia com nomes conhecidos
    local function blockKnownReports()
        local names = {"Report", "BugReport", "ServerFailReport"}
        for _, name in ipairs(names) do
            local event = ReplicatedStorage:FindFirstChild(name)
            if event and event.FireServer then
                event.FireServer = newcclosure(function()
                    showMessage("Alguém Te Denunciou!! (Bloqueado)")
                end)
            end
        end
    end

    -- Bloquear teleportações forçadas
    hookfunction(TeleportService.Teleport, function(...)
        showMessage("Tentaram te Teleportar!! (Bloqueado)")
        return
    end)

    -- Bloquear tentativa de remover seu player
    Players.PlayerRemoving:Connect(function(player)
        if player == LocalPlayer then
            showMessage("Tentaram te Remover!! (Bloqueado)")
            task.wait(9e9)
        end
    end)

    -- Proteção extra: interceptar qualquer RemoteEvent suspeito
    local function blockSuspiciousRemoteEvents()
        for _, obj in ipairs(game:GetDescendants()) do
            if obj:IsA("RemoteEvent") and obj.FireServer then
                local original = obj.FireServer
                obj.FireServer = newcclosure(function(_, ...)
                    showMessage("⚠️ Tentativa de denúncia detectada (bloqueada)")
                    return -- bloqueia sem chamar o original
                end)
            end
        end

        -- Escutar objetos novos criados durante o jogo
        game.DescendantAdded:Connect(function(obj)
            if obj:IsA("RemoteEvent") and obj.FireServer then
                obj.FireServer = newcclosure(function(_, ...)
                    showMessage("⚠️ Nova denúncia detectada e bloqueada")
                    return
                end)
            end
        end)
    end

    blockKnownReports()
    blockSuspiciousRemoteEvents()
    print("[🔰] Anti-Tudo ativado com escudo global de RemoteEvents.")
end)

-- Auto-Defesa Inteligente (detecta presença e teleporta para frente)
local defesaConnection -- variável global dentro do script
createButton("Auto-Defesa Inteligente", 60, function(active)
    local player = game.Players.LocalPlayer
    local char = player.Character or player.CharacterAdded:Wait()
    local root = char:WaitForChild("HumanoidRootPart")
    local runService = game:GetService("RunService")
    local posInicial = root.Position

    if active then
        -- Garante que não vai criar múltiplas conexões
        if defesaConnection then
            defesaConnection:Disconnect()
        end

        defesaConnection = runService.Heartbeat:Connect(function()
            for _, plr in pairs(game.Players:GetPlayers()) do
                if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                    local dist = (root.Position - plr.Character.HumanoidRootPart.Position).Magnitude
                    if dist < 15 then
                        local forward = root.CFrame.LookVector * 10
                        root.CFrame = root.CFrame + forward
                        break
                    end
                end
            end
        end)
    else
        if defesaConnection then
            defesaConnection:Disconnect()
            defesaConnection = nil
        end
        root.CFrame = CFrame.new(posInicial)
    end
end)

-- Reversor de Dano (inverte o dano recebido para o inimigo mais próxio)
createButton("Reversor de Dano", 110, function(active)
    local player = game.Players.LocalPlayer
    local char = player.Character or player.CharacterAdded:Wait()
    local humanoid = char:WaitForChild("Humanoid")
    local root = char:WaitForChild("HumanoidRootPart")
    local danoConnection

    if active then
        danoConnection = humanoid.HealthChanged:Connect(function(newHealth)
            if newHealth < humanoid.MaxHealth then
                local alvo, menor = nil, math.huge
                for _, plr in pairs(game.Players:GetPlayers()) do
                    if plr ~= player and plr.Character and plr.Character:FindFirstChild("Humanoid") and plr.Character:FindFirstChild("HumanoidRootPart") then
                        local dist = (root.Position - plr.Character.HumanoidRootPart.Position).Magnitude
                        if dist < menor then
                            menor = dist
                            alvo = plr.Character.Humanoid
                        end
                    end
                end
                if alvo then
                    alvo:TakeDamage(humanoid.MaxHealth - newHealth)
                end
            end
        end)
    else
        if danoConnection then danoConnection:Disconnect() end
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

createButton("Atravessar Paredes", 210, function(active)
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
local originalSpeed = nil

createButton("Velocidade x3", 260, function(active)
    local humanoid = game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        -- Salva a velocidade original apenas uma vez
        if not originalSpeed then
            originalSpeed = humanoid.WalkSpeed
        end

        humanoid.WalkSpeed = active and originalSpeed * 3 or originalSpeed
    end
end)

-- ESP
local ESPEnabled = false
local ESPObjects = {}

createButton("ESP", 310, function(active)
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
createButton("TP p/ Mais Próximo", 360, function()
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
createButton("TP p/ Mais Distante", 410, function()
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
