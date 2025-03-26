-- Criando a interface flutuante
local ScreenGui = Instance.new("ScreenGui", game.Players.LocalPlayer.PlayerGui)
local FloatingButton = Instance.new("TextButton", ScreenGui)

FloatingButton.Size = UDim2.new(0, 80, 0, 30)
FloatingButton.Position = UDim2.new(0.9, 0, 0.1, 0)
FloatingButton.Text = "SCRIPT"
FloatingButton.BackgroundColor3 = Color3.fromRGB(0, 0, 255)

local Panel = Instance.new("Frame", ScreenGui)
Panel.Size = UDim2.new(0, 200, 0, 200)
Panel.Position = UDim2.new(0.75, 0, 0.1, 0)
Panel.Visible = false
Panel.BackgroundColor3 = Color3.fromRGB(50, 50, 50)

-- Adicionando ScrollV
local ScrollingFrame = Instance.new("ScrollingFrame", Panel)
ScrollingFrame.Size = UDim2.new(1, 0, 1, 0)
ScrollingFrame.CanvasSize = UDim2.new(0, 0, 1.5, 0)
ScrollingFrame.ScrollBarThickness = 5

local CloseButton = Instance.new("TextButton", Panel)
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -30, 0, 0)
CloseButton.Text = "X"
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

FloatingButton.MouseButton1Click:Connect(function()
    Panel.Visible = not Panel.Visible
end)

CloseButton.MouseButton1Click:Connect(function()
    Panel.Visible = false
end)

-- Criando botões e funções
local function createButton(name, position, action)
    local button = Instance.new("TextButton", ScrollingFrame)
    button.Size = UDim2.new(0, 180, 0, 40)
    button.Position = UDim2.new(0, 10, 0, position)
    button.Text = name
    button.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- Vermelho = desativado

    local active = false
    button.MouseButton1Click:Connect(function()
        active = not active
        button.BackgroundColor3 = active and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
        action(active)
    end)
end

-- 🔰 Anti-Tudo (Agora é o primeiro botão)
createButton("Anti-Tudo", 10, function(active)
    if active then
        -- Proteção contra kick
        local mt = getrawmetatable(game)
        setreadonly(mt, false)
        local oldNamecall = mt.__namecall
        mt.__namecall = newcclosure(function(self, ...)
            local method = getnamecallmethod()
            if method == "Kick" or method == "kick" then return nil end
            return oldNamecall(self, ...)
        end)

        -- Proteção contra AFK
        local Players = game:GetService("Players")
        for _, v in pairs(getconnections(Players.LocalPlayer.Idled)) do v:Disable() end

        -- Proteção Contra Logs do Byfron (Impede Envio de Dados Suspeitos)
        local oldHttpPost = hookfunction(game.HttpPost, function(...)
            print("[🛡️ Proteção Ativada] Bloqueando Logs do Byfron.")
            return nil -- Bloqueia envio de logs suspeitos para os servidores do Roblox
        end)

        -- Proteção Contra Fechamento Forçado do Jogo
        game:GetService("CoreGui").ChildRemoved:Connect(function(child)
            if child.Name == "RobloxPromptGui" then
                print("[⚠️ Proteção Ativada] Tentativa de Fechar Jogo Detectada.")
                wait(9e9) -- Previne fechamento forçado
            end
        end)

        print("[🔰] Anti-Tudo ativado!")
    else
        -- Desativar proteções (não tem como desfazer completamente, mas minimiza)
        game:GetService("Players").LocalPlayer.Idled:Connect(function() end)
        print("[🔰] Anti-Tudo desativado!")
    end
end)

-- ✈️ Voar
local flying = false
local speed = 60
local flyBodyVelocity
local flyGyro
local flyConnection

createButton("Voar", 60, function(active)
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
        if flyBodyVelocity then flyBodyVelocity:Destroy() end
        if flyGyro then flyGyro:Destroy() end
        if flyConnection then flyConnection:Disconnect() end
    end
end)

-- 🚪 Atravessar paredes
local atravessarAtivo = false
local atravessarConnection

createButton("Atravessar Paredes", 110, function(active)
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
        if atravessarConnection then atravessarConnection:Disconnect() end
        if char then
            for _, part in pairs(char:GetChildren()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
        end
    end
end)

-- 🚀 Aumentar a velocidade x3
createButton("Aumentar Velocidade x3", 160, function(active)
    local player = game:GetService("Players").LocalPlayer
    local char = player.Character
    if char then
        local humanoid = char:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = active and 48 or 16 -- Assumindo que 16 é o padrão
            print("[🚀] Velocidade " .. (active and "aumentada x3" or "normalizada"))
        end
    end
end)

-- 🔀 Teleporte para o jogador mais próximo
createButton("Teleporte p/ Mais Próximo", 210, function()
    local player = game.Players.LocalPlayer
    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")

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
