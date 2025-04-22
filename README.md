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
createButton("Aumentar Velocidade x3", 210, function(active)
    local player = game:GetService("Players").LocalPlayer
    local char = player.Character
    if char then
        local humanoid = char:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = active and 48 or 16
        end
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
createButton("Teleporte p/ Mais Próximo", 310, function()
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

-- Teleporte para o jogador mais distante
createButton("Teleporte p/ Mais Distante", 360, function()
    local player = game.Players.LocalPlayer
    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")

    if root then
        local farthestPlayer
        local farthestDistance = 0

        for _, otherPlayer in pairs(game.Players:GetPlayers()) do
            if otherPlayer ~= player and otherPlayer.Character then
                local otherRoot = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
                if otherRoot then
                    local distance = (root.Position - otherRoot.Position).Magnitude
                    if distance > farthestDistance then
                        farthestDistance = distance
                        farthestPlayer = otherRoot
                    end
                end
            end
        end

        if farthestPlayer then
            root.CFrame = farthestPlayer.CFrame
        end
    end
end)
