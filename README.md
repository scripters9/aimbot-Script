--// CONFIGURAÇÕES
local FOV_RADIUS = 120
local AIM_PART = "Head"
local DANCE_ANIMATION_ID = "33796059" -- ID animação dança R6

--// SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

--// CORES PARA EFEITO BALADA E ESP
local COLORS = {
    Color3.fromRGB(255, 0, 0),
    Color3.fromRGB(255, 127, 0),
    Color3.fromRGB(255, 255, 0),
    Color3.fromRGB(0, 255, 0),
    Color3.fromRGB(0, 0, 255),
    Color3.fromRGB(75, 0, 130),
    Color3.fromRGB(148, 0, 211),
}

--// UI PRINCIPAL
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.Name = "AimbotUI"

--// EFEITO BALADA (flash colorido por 10s)
do
    local flash = Instance.new("Frame", ScreenGui)
    flash.Size = UDim2.new(1,0,1,0)
    flash.BackgroundColor3 = COLORS[1]
    flash.BackgroundTransparency = 1
    flash.ZIndex = 9999

    local tweenInfo = TweenInfo.new(0.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut)
    local startTime = tick()
    local duration = 10
    local function pulseColor()
        local colorIndex = 1
        while tick() - startTime < duration do
            local nextIndex = colorIndex % #COLORS + 1
            local tweenIn = TweenService:Create(flash, tweenInfo, {BackgroundTransparency = 0.7, BackgroundColor3 = COLORS[colorIndex]})
            local tweenOut = TweenService:Create(flash, tweenInfo, {BackgroundTransparency = 1, BackgroundColor3 = COLORS[nextIndex]})
            tweenIn:Play()
            tweenIn.Completed:Wait()
            tweenOut:Play()
            tweenOut.Completed:Wait()
            colorIndex = nextIndex
        end
        flash:Destroy()
    end
    coroutine.wrap(pulseColor)()
end

--// MENSAGEM ANIMADA
local function showMessage(text, duration, callback)
    local msg = Instance.new("TextLabel", ScreenGui)
    msg.Size = UDim2.new(0, 400, 0, 50)
    msg.Position = UDim2.new(0.5, -200, 0.5, -25)
    msg.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    msg.TextColor3 = Color3.new(1, 1, 1)
    msg.Text = text
    msg.Font = Enum.Font.GothamBold
    msg.TextScaled = true
    msg.BackgroundTransparency = 1
    msg.TextTransparency = 1
    msg.ZIndex = 10000

    local tweenIn = TweenService:Create(msg, TweenInfo.new(1), {
        TextTransparency = 0,
        BackgroundTransparency = 0.2
    })
    local tweenOut = TweenService:Create(msg, TweenInfo.new(1), {
        TextTransparency = 1,
        BackgroundTransparency = 1
    })

    tweenIn:Play()
    tweenIn.Completed:Wait()

    task.delay(duration, function()
        tweenOut:Play()
        tweenOut.Completed:Wait()
        msg:Destroy()
        if callback then callback() end
    end)
end

showMessage("By BaconPurple lol", 3, function()
    showMessage("Valeu por testar", 3)
end)

--// BOTÃO AIMBOT
local aimbotOn = false
local Button = Instance.new("TextButton", ScreenGui)
Button.Size = UDim2.new(0, 130, 0, 45)
Button.Position = UDim2.new(0.05, 0, 0.8, 0)
Button.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Button.Text = "Aimbot: OFF"
Button.TextColor3 = Color3.new(1, 1, 1)
Button.TextScaled = true
Button.Font = Enum.Font.GothamBold
Button.ZIndex = 10000

Button.MouseButton1Click:Connect(function()
    aimbotOn = not aimbotOn
    Button.Text = "Aimbot: " .. (aimbotOn and "ON" or "OFF")
    Button.BackgroundColor3 = aimbotOn and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(30, 30, 30)
end)

--// FOV CIRCLE
local FOV_Circle = Drawing.new("Circle")
FOV_Circle.Color = Color3.fromRGB(0, 255, 0)
FOV_Circle.Thickness = 2
FOV_Circle.Radius = FOV_RADIUS
FOV_Circle.Filled = false
FOV_Circle.Transparency = 0.4
FOV_Circle.Visible = true

RunService.RenderStepped:Connect(function()
    local view = Camera.ViewportSize
    FOV_Circle.Position = Vector2.new(view.X / 2, view.Y / 2)
end)

--// Aimbot Função
local function GetClosestTarget()
    local closest = nil
    local shortest = FOV_RADIUS
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(AIM_PART) then
            local part = player.Character[AIM_PART]
            local screenPoint, onScreen = Camera:WorldToViewportPoint(part.Position)
            if onScreen then
                local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
                local dist = (Vector2.new(screenPoint.X, screenPoint.Y) - screenCenter).Magnitude
                if dist < shortest then
                    shortest = dist
                    closest = player
                end
            end
        end
    end
    return closest
end

RunService.RenderStepped:Connect(function()
    if aimbotOn then
        local target = GetClosestTarget()
        if target and target.Character and target.Character:FindFirstChild(AIM_PART) then
            local pos = target.Character[AIM_PART].Position
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, pos)
        end
    end
end)

--// Animação de Dança R6
local function playDanceAnimation()
    local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    local humanoid = char:WaitForChild("Humanoid", 5)
    if humanoid then
        local anim = Instance.new("Animation")
        anim.AnimationId = "rbxassetid://" .. DANCE_ANIMATION_ID
        local track = humanoid:LoadAnimation(anim)
        track.Looped = true
        track:Play()
        task.delay(10, function() track:Stop() end)
    end
end
playDanceAnimation()

--// ESP Colorido com Barra de Vida
local espTable = {}
local espColorIndex = 1
local espColorTimer = 0
local espColorChangeInterval = 0.6

local function createEsp(player)
    if espTable[player] then return end
    local function setupChar(char)
        if not char then return end
        local head = char:WaitForChild("Head", 5)
        local humanoid = char:WaitForChild("Humanoid", 5)
        if not head or not humanoid then return end

        local box = Drawing.new("Square")
        box.Visible = true
        box.Color = COLORS[1]
        box.Thickness = 2
        box.Transparency = 0.5
        box.Filled = false

        local healthBarBg = Drawing.new("Square")
        healthBarBg.Visible = true
        healthBarBg.Color = Color3.new(0.2, 0.2, 0.2)
        healthBarBg.Thickness = 1
        healthBarBg.Filled = true
        healthBarBg.Transparency = 0.4

        local healthBar = Drawing.new("Square")
        healthBar.Visible = true
        healthBar.Color = COLORS[1]
        healthBar.Thickness = 0
        healthBar.Filled = true
        healthBar.Transparency = 0.5

        espTable[player] = {
            box = box,
            healthBarBg = healthBarBg,
            healthBar = healthBar,
            humanoid = humanoid,
            head = head,
            character = char
        }
    end

    player.CharacterAdded:Connect(function(char)
        task.wait(1)
        setupChar(char)
    end)

    if player.Character then
        setupChar(player.Character)
    end
end

local function removeEsp(player)
    local esp = espTable[player]
    if esp then
        esp.box:Remove()
        esp.healthBarBg:Remove()
        esp.healthBar:Remove()
        espTable[player] = nil
    end
end

for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        createEsp(player)
    end
end

Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        createEsp(player)
    end
end)

Players.PlayerRemoving:Connect(function(player)
    removeEsp(player)
end)

RunService.RenderStepped:Connect(function(dt)
    espColorTimer = espColorTimer + dt
    if espColorTimer >= espColorChangeInterval then
        espColorTimer = 0
        espColorIndex = espColorIndex % #COLORS + 1
        for _, esp in pairs(espTable) do
            esp.box.Color = COLORS[espColorIndex]
            esp.healthBar.Color = COLORS[espColorIndex]
        end
    end

    for player, esp in pairs(espTable) do
        local char = esp.character
        local humanoid = esp.humanoid
        local head = esp.head
        if char and head and humanoid and head.Parent == char then
            local headPos, onScreen = Camera:WorldToViewportPoint(head.Position)
            if onScreen then
                local size = 40
                esp.box.Size = Vector2.new(size, size)
                esp.box.Position = Vector2.new(headPos.X - size / 2, headPos.Y - size / 2)
                esp.box.Visible = true

                local barWidth = size
                local barHeight = 6
                local healthPercent = math.clamp(humanoid.Health / humanoid.MaxHealth, 0, 1)

                esp.healthBarBg.Size = Vector2.new(barWidth, barHeight)
                esp.healthBarBg.Position = Vector2.new(headPos.X - barWidth / 2, headPos.Y - size / 2 - 10)
                esp.healthBarBg.Visible = true

                esp.healthBar.Size = Vector2.new(barWidth * healthPercent, barHeight)
                esp.healthBar.Position = Vector2.new(headPos.X - barWidth / 2, headPos.Y - size / 2 - 10)
                esp.healthBar.Visible = true
            else
                esp.box.Visible = false
                esp.healthBarBg.Visible = false
                esp.healthBar.Visible = false
            end
        end
    end
end)
