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
Panel.Size = UDim2.new(0, 180, 0, 280)
Panel.Position = UDim2.new(0.75, 0, 0.1, 0)
Panel.Visible = false
Panel.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
Panel.Parent = ScreenGui

local ScrollingFrame = Instance.new("ScrollingFrame")
ScrollingFrame.Size = UDim2.new(1, 0, 1, 0)
ScrollingFrame.CanvasSize = UDim2.new(0, 0, 10, 0)
ScrollingFrame.ScrollBarThickness = 5
ScrollingFrame.Parent = Panel

-- Botão X (fechar painel)
local CloseButton = Instance.new("TextButton")
CloseButton.Size       = UDim2.new(0, 30, 0, 30)
CloseButton.Position   = UDim2.new(1, -30, 0, 0)
CloseButton.Text       = "X"
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
CloseButton.Parent     = Panel
CloseButton.Text = "X"

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

-- Auto‑Heal (corrigido)
local healActive   = false
local healThread   = nil

createButton("Auto-Heal", 60, function(active)
healActive = active

if healActive and not healThread then  
    healThread = task.spawn(function()  
        while healActive do  
            local hum = game.Players.LocalPlayer.Character and  
                        game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")  
            if hum and hum.Health > 0 and hum.Health < hum.MaxHealth then  
                hum.Health = math.min(hum.Health + 10, hum.MaxHealth)  
            end  
            task.wait(0.1)  
        end  
    end)  
elseif not healActive and healThread then  
    -- Para a thread: setamos healActive=false; loop vai encerrar,  
    -- depois anulamos a referência  
    healThread = nil  
end

end)

--Barreira Impenetrável
createButton("Barreira Impenetrável", 110, function(active)
    if not active then return end

    local player = game.Players.LocalPlayer
    local humanoid = player.Character and player.Character:WaitForChild("Humanoid")

    local function notificar(msg, cor)
        local gui = Instance.new("ScreenGui", game.CoreGui)
createButton("Barreira Impenetrável", 120, function(active)
    if not active then return end

    local player = game.Players.LocalPlayer
    local humanoid = player.Character and player.Character:WaitForChild("Humanoid")

    local function notificar(msg, cor)
        local gui = Instance.new("ScreenGui", game.CoreGui)
        local lbl = Instance.new("TextLabel", gui)
        lbl.Size = UDim2.new(0, 500, 0, 40)
        lbl.Position = UDim2.new(0.5, -250, 0.05, 0)
        lbl.Text = msg
        lbl.TextColor3 = cor or Color3.fromRGB(255, 255, 0)
        lbl.BackgroundTransparency = 1
        lbl.TextScaled = true
        wait(2)
        gui:Destroy()
    end

    local function criarBarreira()
        local char = player.Character
        if not char then return end

        -- Cria a barreira semi-transparente
        local barreira = Instance.new("Part")
        barreira.Name = "Barreira"
        barreira.Shape = Enum.PartType.Ball
        barreira.Size = Vector3.new(20, 20, 20)
        barreira.Transparency = 0.5
        barreira.Material = Enum.Material.SmoothPlastic
        barreira.Color = Color3.fromRGB(0, 0, 255)
        barreira.CanCollide = true
        barreira.Anchored = false  -- Para que a barreira siga o jogador
        barreira.Parent = char
        barreira.CFrame = char.HumanoidRootPart.CFrame

        -- Criar a barreira ao redor do jogador e mover com ele
        local weld = Instance.new("WeldConstraint", barreira)
        weld.Part0 = barreira
        weld.Part1 = char.HumanoidRootPart

        -- Função para bloquear ataques físicos (como espadas)
        local function bloquearAtaquesFisicos()
            for _, obj in pairs(workspace:GetChildren()) do
                if obj:IsA("Part") and obj.Parent and obj.Parent:IsA("Player") and obj.Parent ~= player then
                    -- Verifica se o objeto que colide com a barreira é uma arma (por exemplo, espada)
                    if barreira and barreira:IsPointInRegion(obj.Position) then
                        -- Se o objeto for uma espada ou arma, bloqueia o ataque
                        if obj.Name:lower():find("sword") or obj.Name:lower():find("weapon") then
                            obj:Destroy()  -- Destruir a espada ou arma para impedir o ataque
                            notificar("ATAQUE BLOQUEADO: ARMA DE CORPO A CORPO", Color3.fromRGB(255, 0, 0))
                        end
                    end
                end
            end
        end

        -- Conectar a função que bloqueia interações físicas
        game:GetService("RunService").Heartbeat:Connect(bloquearAtaquesFisicos)

        -- Regenerar vida quando a barreira for ativada
        local vidaPerdida = humanoid.Health
        humanoid.HealthChanged:Connect(function()
            if humanoid.Health < vidaPerdida then
                humanoid.Health = vidaPerdida
                notificar("VIDA RESTAURADA!", Color3.fromRGB(0, 255, 0))
            end
        end)

        return barreira
    end

    -- Ação de ativar/desativar
    local barreiraAtiva
    if not barreiraAtiva then
        barreiraAtiva = criarBarreira()
        notificar("BARRERA IMPENETRÁVEL ATIVADA", Color3.fromRGB(0, 0, 255))
    else
        if barreiraAtiva then
            barreiraAtiva:Destroy()
            barreiraAtiva = nil
            notificar("BARRERA IMPENETRÁVEL DESATIVADA", Color3.fromRGB(255, 0, 0))
        end
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

-- Velocidade x3 (corrigido)
local speedActive   = false
local originalSpeed = nil      -- guardará a velocidade padrão

createButton("Velocidade x3", 260, function(active)
local hum = game.Players.LocalPlayer.Character and
game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
if not hum then return end

speedActive = active  
if speedActive then  
    -- guarda só primeira vez  
    originalSpeed = originalSpeed or hum.WalkSpeed  
    hum.WalkSpeed = originalSpeed * 3          -- triplica  
else  
    if originalSpeed then  
        hum.WalkSpeed = originalSpeed          -- volta ao normal  
    end  
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

