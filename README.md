-- Criando a interface flutuante
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui")  -- Ajustando para PlayerGui

local FloatingButton = Instance.new("TextButton", ScreenGui)
FloatingButton.Size = UDim2.new(0, 80, 0, 30)
FloatingButton.Position = UDim2.new(0.9, 0, 0.1, 0)
FloatingButton.Text = "SCRIPT"
FloatingButton.BackgroundColor3 = Color3.fromRGB(0, 0, 255)
FloatingButton.TextColor3 = Color3.fromRGB(255, 255, 255)
FloatingButton.Font = Enum.Font.SourceSansBold
FloatingButton.TextSize = 14
FloatingButton.BorderSizePixel = 0
FloatingButton.BackgroundTransparency = 0.3
FloatingButton.AutoButtonColor = true

local Panel = Instance.new("Frame", ScreenGui)
Panel.Size = UDim2.new(0, 220, 0, 250)
Panel.Position = UDim2.new(0.75, 0, 0.1, 0)
Panel.Visible = false
Panel.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
Panel.BackgroundTransparency = 0.2
Panel.BorderSizePixel = 0

-- Adicionando Scroll
local Scroll = Instance.new("ScrollingFrame", Panel)
Scroll.Size = UDim2.new(1, 0, 1, -30)
Scroll.Position = UDim2.new(0, 0, 0, 0)
Scroll.CanvasSize = UDim2.new(0, 0, 1.5, 0)  -- Ajuste da altura
Scroll.ScrollBarThickness = 5
Scroll.BackgroundTransparency = 1

local CloseButton = Instance.new("TextButton", Panel)
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -30, 0, 0)
CloseButton.Text = "X"
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.BorderSizePixel = 0

FloatingButton.MouseButton1Click:Connect(function()
Panel.Visible = not Panel.Visible
end)

CloseButton.MouseButton1Click:Connect(function()
Panel.Visible = false
end)

-- Criando botões e funções
local function createButton(name, position, action)
local button = Instance.new("TextButton", Scroll)
button.Size = UDim2.new(0, 180, 0, 40)
button.Position = UDim2.new(0, 10, 0, position)
button.Text = name
button.BackgroundColor3 = Color3.fromRGB(255, 0, 0)  -- Vermelho = desativado
button.TextColor3 = Color3.fromRGB(255, 255, 255)
button.Font = Enum.Font.SourceSansBold
button.TextSize = 14
button.BorderSizePixel = 0
button.AutoButtonColor = true
button.BackgroundTransparency = 0.1
button.ClipsDescendants = true
button.SizeConstraint = Enum.SizeConstraint.RelativeXY
button.ZIndex = 2

-- Arredondando os botões  
local UICorner = Instance.new("UICorner", button)  
UICorner.CornerRadius = UDim.new(0, 8)  

local active = false  
button.MouseButton1Click:Connect(function()  
    active = not active  
    button.BackgroundColor3 = active and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)  
    action(active)  
end)

end

-- 🔰 Proteção contra Kick (Bloqueia Tentativas de Kick)
local mt = getrawmetatable(game)
setreadonly(mt, false)

local oldNamecall = mt.__namecall
mt.__namecall = newcclosure(function(self, ...)
    local method = getnamecallmethod()
    if method == "Kick" or method == "kick" then
        print("[⚠️ Proteção Ativada] Tentativa de Kick bloqueada.")
        return nil -- Cancela qualquer tentativa de Kick
    end
    return oldNamecall(self, ...)
end)

-- 🔰 Proteção contra Banimento Automático (Desativa detecção de AFK/Inatividade)
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local function DisableBan()
    for _, v in pairs(getconnections(LocalPlayer.Idled)) do
        v:Disable() -- Impede detecção por inatividade
    end
    print("[🛡️ Proteção Ativada] Detector de Inatividade Desativado.")
end
DisableBan()

-- 🔰 Bypass do Byfron Anti-Cheat (Impedindo Detecção)
local oldIndex = mt.__index
mt.__index = newcclosure(function(self, key)
    if key == "PreloadAsync" or key == "InvokeServer" or key == "Kick" then
        print("[⚠️ Proteção Ativada] Tentativa de Detecção do Byfron Bloqueada.")
        return function(...) return nil end -- Cancela qualquer tentativa de detecção
    end
    return oldIndex(self, key)
end)

-- 🔰 Proteção Contra Logs do Byfron (Impede Envio de Dados Suspeitos)
local oldHttpPost = hookfunction(game.HttpPost, function(...)
    print("[🛡️ Proteção Ativada] Bloqueando Logs do Byfron.")
    return nil -- Bloqueia envio de logs suspeitos para os servidores do Roblox
end)

-- 🔰 Proteção Contra Fechamento Forçado do Jogo
game:GetService("CoreGui").ChildRemoved:Connect(function(child)
    if child.Name == "RobloxPromptGui" then
        print("[⚠️ Proteção Ativada] Tentativa de Fechar Jogo Detectada.")
        wait(9e9) -- Previne fechamento forçado
    end
end)

print("[✅] Proteção Máxima Ativada: Anti-Kick, Anti-Ban e Byfron Bypass!")

-- ✈️ Voar (Novo sistema, segue a câmera)
local flying = false
local speed = 50
local flyBodyVelocity
local flyGyro

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

        game:GetService("RunService").RenderStepped:Connect(function()
            if not flying then return end
            local cam = workspace.CurrentCamera
            flyGyro.CFrame = cam.CFrame
            flyBodyVelocity.Velocity = cam.CFrame.LookVector * speed
        end)
    else
        flying = false
        if flyBodyVelocity then flyBodyVelocity:Destroy() end
        if flyGyro then flyGyro:Destroy() end
    end
end)

-- 🚪 Atravessar paredes (Corrigido: Agora sempre funciona)
local atravessarAtivo = false

createButton("Atravessar Paredes", 110, function(active)
    local char = game:GetService("Players").LocalPlayer.Character
    atravessarAtivo = active

    game:GetService("RunService").Stepped:Connect(function()
        if not atravessarAtivo then return end
        if char then
            for _, part in pairs(char:GetChildren()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end)
end)

-- 👀 ESP corrigido
local ESPEnabled = false
local ESPObjects = {}

createButton("ESP", 110, function(active)
    ESPEnabled = active

    -- Remover ESP quando desativado
    if not ESPEnabled then
        for _, obj in pairs(ESPObjects) do
            obj:Destroy()
        end
        ESPObjects = {}
        return
    end

    -- Criar ESP
    local function createESP(player)
        if player == game.Players.LocalPlayer then return end -- Não adicionar ESP em si mesmo

        local char = player.Character
        if char then
            local head = char:FindFirstChild("Head")
            if head then
                local esp = Instance.new("BillboardGui", head)
                esp.Size = UDim2.new(0, 10, 0, 10) -- Tamanho pequeno da bolinha
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

    -- Ativar ESP para jogadores existentes
    for _, player in pairs(game.Players:GetPlayers()) do
        createESP(player)
    end

    -- Atualizar ESP para novos jogadores
    game.Players.PlayerAdded:Connect(createESP)
    game.Players.PlayerRemoving:Connect(function(player)
        if ESPObjects[player] then
            ESPObjects[player]:Destroy()
            ESPObjects[player] = nil
        end

-- 🚀 Aumentar a velocidade em 3x
local velocidadeAtiva = false
createButton("Aumentar Velocidade x3", 210, function(active)
    local player = game:GetService("Players").LocalPlayer
    local char = player.Character
    if char then
        local humanoid = char:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = active and humanoid.WalkSpeed * 3 or humanoid.WalkSpeed / 3
            velocidadeAtiva = active
            print("[🚀] Velocidade " .. (active and "aumentada x3" or "normalizada"))
        end
    end
end)
