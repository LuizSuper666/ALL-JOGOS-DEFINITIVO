-- Criando a interface flutuante
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PainelTop"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game:GetService("CoreGui")

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

local ScrollingFrame = Instance.new("ScrollingFrame", Panel)
ScrollingFrame.Size = UDim2.new(1, 0, 1, 0)
ScrollingFrame.CanvasSize = UDim2.new(0, 0, 3, 0)
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

local function createButton(name, position, action)
    local button = Instance.new("TextButton", ScrollingFrame)
    button.Size = UDim2.new(0, 180, 0, 40)
    button.Position = UDim2.new(0, 10, 0, position)
    button.Text = name
    button.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

    local active = false
    button.MouseButton1Click:Connect(function()
        active = not active
        button.BackgroundColor3 = active and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
        action(active)
    end)
end

-- Anti-Tudo Melhorado
createButton("Anti-Tudo", 10, function(active)
    if active then
        -- Proteção contra Kick
        local mt = getrawmetatable(game)
        setreadonly(mt, false)
        local oldNamecall = mt.__namecall
        mt.__namecall = newcclosure(function(self, ...)
            local method = getnamecallmethod()
            if method:lower() == "kick" then return nil end
            return oldNamecall(self, ...)
        end)

        -- Impede desconexão por inatividade
        for _, v in pairs(getconnections(game:GetService("Players").LocalPlayer.Idled)) do
            v:Disable()
        end

        -- Bloqueia logs HTTP (Byfron)
        local oldHttpPost = hookfunction(game.HttpPost, function(...)
            return nil
        end)

        -- Previne fechamento do jogo
        game:GetService("CoreGui").ChildRemoved:Connect(function(child)
            if child.Name == "RobloxPromptGui" then
                wait(9e9)
            end
        end)

        -- Alerta visual
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

        -- Bloqueia denúncias e relatórios
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

-- MODO INTOCÁVEL SUPREMO
local intocavelAtivado = false
local connections = {}

local function protegerPersonagem()
    local player = game:GetService("Players").LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:FindFirstChildOfClass("Humanoid")

    -- Torna intocável fisicamente
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanTouch = false
            part.CanQuery = false
            part.Massless = true
            part:SetNetworkOwnershipAuto()
            -- Ignora física de ragdoll/push
            part.CustomPhysicalProperties = PhysicalProperties.new(0, 0, 0)
        end
    end

    -- Congela valores como Health, WalkSpeed e etc
    if humanoid then
        table.insert(connections, humanoid:GetPropertyChangedSignal("Health"):Connect(function()
            if intocavelAtivado then humanoid.Health = humanoid.MaxHealth end
        end))
        table.insert(connections, humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
            if intocavelAtivado then humanoid.WalkSpeed = 16 end
        end))
        table.insert(connections, humanoid:GetPropertyChangedSignal("JumpPower"):Connect(function()
            if intocavelAtivado then humanoid.JumpPower = 50 end
        end))
        humanoid.BreakJointsOnDeath = false
        humanoid.PlatformStand = false
    end

    -- Apaga scripts maliciosos que grudarem
    for _, obj in ipairs(character:GetDescendants()) do
        if obj:IsA("Script") or obj:IsA("LocalScript") then
            pcall(function() obj:Destroy() end)
        end
    end
end

local function bloquearAtaquesRemotos()
    local mt = getrawmetatable(game)
    local old = mt.__namecall
    setreadonly(mt, false)
    mt.__namecall = newcclosure(function(self, ...)
        local args = {...}
        local method = getnamecallmethod()

        if intocavelAtivado and method == "FireServer" and typeof(self) == "Instance" then
            local nome = self.Name:lower()
            if nome:find("damage") or nome:find("hit") or nome:find("explosion") or nome:find("blast") or nome:find("attack") or nome:find("kill") or nome:find("stun") or nome:find("ragdoll") or nome:find("burn") or nome:find("push") then
                warn("[⚠️] Remote de ataque bloqueado:", self:GetFullName())
                return
            end
        end

        return old(self, unpack(args))
    end)
    setreadonly(mt, true)
end

local function ativarIntocavel()
    protegerPersonagem()
    bloquearAtaquesRemotos()
end

local function desativarIntocavel()
    local player = game:GetService("Players").LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()

    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanTouch = true
            part.CanQuery = true
            part.Massless = false
            part.CustomPhysicalProperties = PhysicalProperties.new()
        end
    end

    for _, conn in ipairs(connections) do
        pcall(function() conn:Disconnect() end)
    end
    connections = {}
end

-- Botão Intocável Supremo
createButton("Intocável", 60, function(active)
    intocavelAtivado = active
    if active then
        ativarIntocavel()
        print("[🛡️] Modo Intocável SUPREMO ativado!")
    else
        desativarIntocavel()
        print("[🛡️] Modo Intocável desativado!")
    end
end)

-- Reaplica ao renascer
game:GetService("Players").LocalPlayer.CharacterAdded:Connect(function()
    wait(1)
    if intocavelAtivado then
        ativarIntocavel()
    end
end)

-- Auto-Heal
createButton("Auto-Heal", 110, function(active)
    local humanoid = game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if active and humanoid then
        spawn(function()
            while active and humanoid and humanoid.Health > 0 do
                wait(0.1)
                if humanoid.Health < humanoid.MaxHealth then
                    humanoid.Health = math.min(humanoid.Health + 1, humanoid.MaxHealth)
                end
            end
        end)
    end
end)

-- Escudo Refletor Supremo
createButton("ESCUDO REFLETOR SUPREMO V2", 360, function(active)
    if not active then return end

    local player = game:GetService("Players").LocalPlayer

    -- Notificação na tela
    local function notificar(txt, cor)
        local gui = Instance.new("ScreenGui", game.CoreGui)
        gui.Name = "NotificacaoRefletor"
        local label = Instance.new("TextLabel", gui)
        label.Size = UDim2.new(0, 500, 0, 40)
        label.Position = UDim2.new(0.5, -250, 0.05, 0)
        label.Text = txt
        label.TextColor3 = cor or Color3.fromRGB(255, 255, 0)
        label.TextStrokeTransparency = 0
        label.BackgroundTransparency = 1
        label.TextScaled = true
        wait(2)
        gui:Destroy()
    end

    -- Refletir o dano
    local function refletir(origem, dano)
        if origem and origem.Character and origem.Character:FindFirstChild("Humanoid") then
            local humanoid = origem.Character:FindFirstChild("Humanoid")
            humanoid:TakeDamage(dano or 100)
            notificar("REFLETIDO!", Color3.fromRGB(255, 0, 0))
            warn("[⚔️] Dano refletido para:", origem.Name)
        end
    end

    -- Lista completa de comandos perigosos
    local blockedWords = {
        -- Comandos clássicos de punição
        "kick", "ban", "kill", "explode", "punish", "jail", "reset", "break", "burn", "freeze",
        "lag", "slow", "crash", "trap", "void", "voidkill", "arrest", "eject", "loopkill",

        -- Comandos troll ou visuais
        "fling", "spin", "smash", "yeet", "nuke", "smite", "drag", "dragdown", "launch", "confuse",
        "control", "float", "seize", "blind", "mute", "explodehead", "shrink", "grow", "invert",
        "disorient", "bounce", "wave", "forcefield", "gravity", "poop", "vomit", "fart", "pee",
        "danceforcibly", "dancefreeze", "stun", "shock", "snare", "sit", "kneel", "strip", "infect",
        "glitch", "trapkill", "collapse", "bald", "ghostify", "removeclothes", "spinhead", "launchup",
        "explodelegs", "explodearms", "swap", "telekill", "messup", "crush", "shatter", "electrocute",
        "whirl", "swirl", "suffocate", "fade", "disappear", "explodebody", "demolish", "disassemble",
        "pulverize", "flatten", "implode", "explodeface", "vomitloop", "burnloop", "pooploop", "spinloop"
    }

    -- Proteção contra remotes e comandos
    local mt = getrawmetatable(game)
    local old = mt.__namecall
    setreadonly(mt, false)

    mt.__namecall = newcclosure(function(self, ...)
        local args = {...}
        local method = getnamecallmethod()

        if method == "FireServer" or method == "InvokeServer" then
            local remoteName = self.Name:lower()
            for _, word in ipairs(blockedWords) do
                if remoteName:find(word) then
                    notificar("ATAQUE BLOQUEADO: " .. word:upper())
                    for _, v in pairs(args) do
                        if typeof(v) == "Instance" and v:IsA("Player") and v ~= player then
                            refletir(v, 100)
                        end
                    end
                    return nil
                end
            end

            for _, arg in ipairs(args) do
                if typeof(arg) == "string" then
                    for _, word in ipairs(blockedWords) do
                        if arg:lower():find(word) then
                            notificar("COMANDO BLOQUEADO: " .. word:upper())
                            return nil
                        end
                    end
                end
            end
        elseif method == "Kick" then
            notificar("TENTATIVA DE KICK BLOQUEADA", Color3.fromRGB(255, 100, 100))
            return nil
        end

        return old(self, unpack(args))
    end)

    setreadonly(mt, true)

    -- Proteção contra dano fatal
    local humanoid = player.Character and player.Character:WaitForChild("Humanoid")
    humanoid.BreakJointsOnDeath = false
    humanoid:GetPropertyChangedSignal("Health"):Connect(function()
        if humanoid.Health <= 1 then
            humanoid.Health = humanoid.MaxHealth
            notificar("DANO FATAL BLOQUEADO", Color3.fromRGB(255, 0, 0))
        end
    end)

    -- Observa pastas para detectar scripts perigosos
    local function observarPasta(pasta)
        pasta.ChildAdded:Connect(function(child)
            local name = child.Name:lower()
            for _, palavra in ipairs(blockedWords) do
                if name:find(palavra) then
                    notificar("AÇÃO DE ADMIN DETECTADA: " .. palavra:upper(), Color3.fromRGB(255, 150, 0))
                    wait(0.2)
                    child:Destroy()
                end
            end
        end)
    end

    observarPasta(game:GetService("Workspace"))
    observarPasta(game:GetService("ReplicatedStorage"))
    observarPasta(game:GetService("StarterGui"))
    observarPasta(game:GetService("CoreGui"))

    -- Monitora comandos no chat
    game.Players.LocalPlayer.Chatted:Connect(function(msg)
    local lower = msg:lower()
    for _, palavra in ipairs(blockedWords) do
        if lower:find("/" .. palavra) or lower:find(":" .. palavra) or lower:find(";" .. palavra) then
            notificar("COMANDO NO CHAT BLOQUEADO: " .. palavra:upper(), Color3.fromRGB(255, 200, 0))
        end
    end
end)

    notificar("ESCUDO REFLETOR SUPREMO ATIVADO!", Color3.fromRGB(0, 255, 0))
end)

-- Voar (Corrigido)
local flying = false
local speed = 60
local flyBodyVelocity
local flyGyro
local flyConnection

createButton("Voar", 110, function(active)
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
createButton("Velocidade x3", 160, function(active)
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
createButton("TP p/ Mais Próximo", 210, function()
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
createButton("TP p/ Mais Distante", 260, function()
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
