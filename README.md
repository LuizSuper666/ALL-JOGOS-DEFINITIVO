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
	button.Size = UDim2.new(0, 160, 0, 40)
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
    local screenGui = Instance.new("ScreenGui")
    screenGui.Parent = game:GetService("CoreGui")

    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(0, 400, 0, 100)
    textLabel.Position = UDim2.new(0.5, -200, 0, 10)
    textLabel.Text = message
    textLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
    textLabel.TextSize = 30
    textLabel.BackgroundTransparency = 1
    textLabel.Parent = screenGui

    task.delay(2, function()
        screenGui:Destroy()
    end)
end

-- Exemplo de uso:
-- showMessage("Alerta: Algo aconteceu!")

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

---- Desvio Automático 
createButton("Desvio Projétil", 60, function(active) local RunService = game:GetService("RunService") local player = game.Players.LocalPlayer local char = player.Character or player.CharacterAdded:Wait() local humanoidRootPart = char:WaitForChild("HumanoidRootPart") local moveLeft = true local conn

if active then
	conn = RunService.Heartbeat:Connect(function()
		for _, obj in pairs(workspace:GetDescendants()) do
			if obj:IsA("BasePart") and obj.Velocity.Magnitude > 100 and obj.Position:FuzzyEq(humanoidRootPart.Position, 10) == false then
				local dir = moveLeft and Vector3.new(-5, 0, 0) or Vector3.new(5, 0, 0)
				humanoidRootPart.CFrame = humanoidRootPart.CFrame + dir
				moveLeft = not moveLeft
				break
			end
		end
	end)
else
	if conn then conn:Disconnect() end
end

end)

-- Botão 2: Predição Adaptável
createButton("Predição Adaptável", 110, function(active) local RunService = game:GetService("RunService") local player = game.Players.LocalPlayer local camera = workspace.CurrentCamera local points = {} local lastPos = nil local conn

local function clearPoints()
	for _, p in pairs(points) do
		p:Destroy()
	end
	points = {}
end

if active then
	conn = RunService.Heartbeat:Connect(function()
		local target = nil
		for _, plr in pairs(game.Players:GetPlayers()) do
			if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
				target = plr.Character.HumanoidRootPart
				break
			end
		end

		if target then
			local vel = target.Velocity
			local currentPos = target.Position

			-- Verifica movimento brusco
			if lastPos and (lastPos - currentPos).Magnitude > 3 then
				clearPoints()
			end

			lastPos = currentPos

			clearPoints()

			for i = 0.5, 2, 0.5 do
				local predicted = currentPos + vel * i
				local part = Instance.new("Part")
				part.Size = Vector3.new(0.4, 0.4, 0.4)
				part.Anchored = true
				part.CanCollide = false
				part.Material = Enum.Material.Neon
				part.Color = Color3.fromRGB(0, 255, 255)
				part.Position = predicted
				part.Parent = workspace  -- Aqui está a correção, agora a parte é paiada no workspace
				table.insert(points, part)
			end
		end
	end)
else
	if conn then conn:Disconnect() end
	clearPoints()  -- Limpa os pontos quando desativado
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
