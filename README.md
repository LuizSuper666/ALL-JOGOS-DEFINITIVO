-- Criando a interface flutuante
local ScreenGui = Instance.new("ScreenGui", game.Players.LocalPlayer.PlayerGui) -- Mudança para PlayerGui
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

-- Adicionando ScrollV
local ScrollingFrame = Instance.new("ScrollingFrame", Panel)
ScrollingFrame.Size = UDim2.new(1, 0, 1, -30)
ScrollingFrame.Position = UDim2.new(0, 0, 0, 30)
ScrollingFrame.CanvasSize = UDim2.new(0, 0, 2, 0) -- Ajustar conforme mais botões forem adicionados
ScrollingFrame.ScrollBarThickness = 5

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

-- 🔰 Anti-Tudo
createButton("Anti-Tudo", 10, function(active)
    if active then
        local mt = getrawmetatable(game)
        setreadonly(mt, false)
        local oldNamecall = mt.__namecall
        mt.__namecall = newcclosure(function(self, ...)
            local method = getnamecallmethod()
            if method == "Kick" or method == "kick" then return nil end
            return oldNamecall(self, ...)
        end)

        local Players = game:GetService("Players")
        for _, v in pairs(getconnections(Players.LocalPlayer.Idled)) do v:Disable() end

        local oldHttpPost = hookfunction(game.HttpPost, function(...)
            print("[🛡️ Proteção Ativada] Bloqueando Logs do Byfron.")
            return nil
        end)

        game:GetService("CoreGui").ChildRemoved:Connect(function(child)
            if child.Name == "RobloxPromptGui" then
                print("[⚠️ Proteção Ativada] Tentativa de Fechar Jogo Detectada.")
                wait(9e9)
            end
        end)
    else
        game:GetService("Players").LocalPlayer.Idled:Connect(function() end)
    end
end)

-- ✈️ Voar
createButton("Voar", 60, function(active)
    local player = game:GetService("Players").LocalPlayer
    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")

    if active and root then
        local flyBodyVelocity = Instance.new("BodyVelocity", root)
        flyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
        flyBodyVelocity.MaxForce = Vector3.new(4000, 4000, 4000)

        local flyGyro = Instance.new("BodyGyro", root)
        flyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
        flyGyro.CFrame = root.CFrame

        game:GetService("RunService").RenderStepped:Connect(function()
            if not active then return end
            local cam = workspace.CurrentCamera
            flyGyro.CFrame = cam.CFrame
            flyBodyVelocity.Velocity = cam.CFrame.LookVector * 60
        end)
    end
end)

-- 🚀 Aumentar Velocidade x3
createButton("Aumentar Velocidade x3", 110, function(active)
    local player = game:GetService("Players").LocalPlayer
    local char = player.Character
    if char then
        local humanoid = char:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = active and 48 or 16 -- Padrão do Roblox é 16
            print("[🚀] Velocidade " .. (active and "aumentada x3" or "normalizada"))
        end
    end
end)

-- 🚪 Atravessar Paredes
createButton("Atravessar Paredes", 160, function(active)
    local char = game:GetService("Players").LocalPlayer.Character
    game:GetService("RunService").Stepped:Connect(function()
        if not active then return end
        if char then
            for _, part in pairs(char:GetChildren()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end
    end)
end)

-- 👀 ESP
createButton("ESP", 210, function(active)
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer then
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
                    dot.BorderSizePixel = 0
                end
            end
        end
    end
end)

-- 📍 Teleporte para o personagem mais próximo
createButton("Teleporte Perto", 260, function(active)
    if active then
        local player = game.Players.LocalPlayer
        local char = player.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")

        if root then
            local closestPlayer
            local closestDistance = math.huge

            for _, otherPlayer in pairs(game.Players:GetPlayers()) do
                if otherPlayer ~= player then
                    local otherChar = otherPlayer.Character
                    local otherRoot = otherChar and otherChar:FindFirstChild("HumanoidRootPart")
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
                root.CFrame = closestPlayer.CFrame + Vector3.new(2, 0, 2)
            end
        end
    end
end)
