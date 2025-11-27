-- Painel BIGBIG - AIMBOT/ESP com highlight e círculo FOV vermelho claro
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local fov = 120
local headAimEnabled = true
local teamCheck = false
local showFovCircle = false
local aimKey = Enum.UserInputType.MouseButton2
local aimMode = "NONE" -- "FULL" ou "LEGIT"
local espEnabled = false
local legitSmooth = 0.15

-- Vermelho vibrante
local fovColor   = Color3.fromRGB(255,80,80)
local espColor   = Color3.fromRGB(255,80,80)
local outlineESP = Color3.fromRGB(55,24,24)

----------------------------------------
-- UI helpers
local function round(obj, val) local ui=Instance.new("UICorner") ui.CornerRadius=UDim.new(0, val or 9) ui.Parent=obj end
local function shadow(obj, t)
    local s=Instance.new("ImageLabel") s.BackgroundTransparency=1 s.Image="rbxassetid://5028857084"
    s.ImageColor3=Color3.new(0,0,0) s.ScaleType=Enum.ScaleType.Slice s.SliceCenter=Rect.new(24,24,276,276)
    s.Size=UDim2.new(1,10,1,10) s.Position=UDim2.new(0,-5,0,-5) s.ZIndex=obj.ZIndex-1 s.Parent=obj s.ImageTransparency=t or 0.55
end

----------------------------------------
-- Painel principal
local gui = Instance.new("ScreenGui")
gui.Name = "BIGBIG_AimbotESP_GUI"
gui.ResetOnSpawn = false
gui.Parent = game:GetService("CoreGui")

local frame = Instance.new("Frame", gui)
frame.Position = UDim2.new(0,30,0,120)
frame.Size = UDim2.new(0,225,0,232)
frame.BackgroundColor3 = Color3.fromRGB(31,20,32)
frame.BorderSizePixel = 0 frame.ZIndex=2 frame.Active=true frame.Draggable=true
round(frame,13) shadow(frame,0.4)

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,24)
title.Position = UDim2.new(0,0,0,2)
title.Text = "🧠 BIGBIG"
title.TextColor3 = fovColor
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 17
title.ZIndex = 5

local collapseBtn = Instance.new("TextButton", frame)
collapseBtn.Text = "—"
collapseBtn.Font = Enum.Font.GothamBold
collapseBtn.TextSize = 15
collapseBtn.Size = UDim2.new(0,23,0,21)
collapseBtn.Position = UDim2.new(1, -26, 0, 3)
collapseBtn.BackgroundColor3 = Color3.fromRGB(45,32,64)
collapseBtn.TextColor3 = fovColor
collapseBtn.BorderSizePixel = 0
collapseBtn.ZIndex = 8
round(collapseBtn,7)

local iconBtn = Instance.new("TextButton", gui)
iconBtn.Name = "OpenPanelBtn"
iconBtn.Visible = false
iconBtn.Size = UDim2.new(0,30,0,30)
iconBtn.Position = frame.Position + UDim2.new(0,8,0,0)
iconBtn.BackgroundColor3 = Color3.fromRGB(46,38,61)
iconBtn.BackgroundTransparency = 0
iconBtn.BorderSizePixel = 0
iconBtn.Text = "🧠"
iconBtn.TextColor3 = fovColor
iconBtn.TextSize = 20
iconBtn.Font = Enum.Font.GothamBold
iconBtn.ZIndex = 12
round(iconBtn,8) shadow(iconBtn,0.36)

local lastPanelPos = frame.Position
collapseBtn.MouseButton1Click:Connect(function()
    lastPanelPos = frame.Position
    frame.Visible = false
    iconBtn.Position = lastPanelPos + UDim2.new(0,8,0,0)
    iconBtn.Visible = true
end)
iconBtn.MouseButton1Click:Connect(function()
    frame.Visible = true
    frame.Position = iconBtn.Position - UDim2.new(0,8,0,0)
    iconBtn.Visible = false
end)

-- Botão-rádio
local function makeRadio(label, pos, isOn, setF, color)
    color = color or Color3.fromRGB(95,185,255)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.46,0,0,25)
    btn.Position = pos
    btn.BackgroundColor3 = isOn() and color or Color3.fromRGB(45,40,54)
    btn.BorderSizePixel = 0
    btn.Text = label
    btn.TextColor3 = isOn() and Color3.new(1,1,1) or Color3.fromRGB(200,190,255)
    btn.TextSize = 14
    btn.Font = Enum.Font.GothamBold
    btn.Parent = frame
    btn.ZIndex = 4
    round(btn,6)
    btn.MouseButton1Click:Connect(function()
        setF()
        btn.BackgroundColor3 = isOn() and color or Color3.fromRGB(45,40,54)
        btn.TextColor3 = isOn() and Color3.fromRGB(255,255,255) or Color3.fromRGB(200,190,255)
    end)
    return btn
end

local function makeToggle(label, pos, isOn, setF, color)
    color = color or Color3.fromRGB(105,255,140)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.48,0,0,24)
    btn.Position = pos
    btn.BackgroundColor3 = isOn() and color or Color3.fromRGB(45,40,54)
    btn.BorderSizePixel = 0
    btn.Text = label
    btn.TextColor3 = Color3.new(1,1,1)
    btn.TextSize = 11
    btn.Font = Enum.Font.GothamBold
    btn.Parent = frame
    btn.ZIndex = 3
    round(btn,6)
    btn.MouseButton1Click:Connect(function()
        setF()
        btn.BackgroundColor3 = isOn() and color or Color3.fromRGB(45,40,54)
    end)
    return btn
end

-- AIM FULL / LEGIT lado a lado
makeRadio("AIM FULL",UDim2.new(0.04,0,0,32), function() return aimMode=="FULL" end,
    function() aimMode = (aimMode=="FULL") and "NONE" or "FULL" end, Color3.fromRGB(255,100,120)
)
makeRadio("AIM LEGIT",UDim2.new(0.52,0,0,32), function() return aimMode=="LEGIT" end,
    function() aimMode = (aimMode=="LEGIT") and "NONE" or "LEGIT" end, Color3.fromRGB(90,175,255)
)
-- ESP / Show FOV
makeToggle("ESP",UDim2.new(0.04,0,0,63), function() return espEnabled end,function() espEnabled = not espEnabled end)
makeToggle("Ver FOV",UDim2.new(0.52,0,0,63), function() return showFovCircle end,function() showFovCircle = not showFovCircle if fovCircle then fovCircle.Visible = showFovCircle end end,Color3.fromRGB(255,140,140))
-- Head/Body / Team
local alvoBtn = makeToggle("Alvo: Head",UDim2.new(0.04,0,0,93),function() return headAimEnabled end,
    function() headAimEnabled = not headAimEnabled alvoBtn.Text = "Alvo: "..(headAimEnabled and "Head" or "Body") end,Color3.fromRGB(255,205,200)
)
makeToggle("TeamChk",UDim2.new(0.52,0,0,93), function() return teamCheck end,function() teamCheck = not teamCheck end,Color3.fromRGB(115,205,255))

-- FOV e botões
local fovLabel = Instance.new("TextLabel", frame)
fovLabel.Position = UDim2.new(0.09,0,0,130)
fovLabel.Size = UDim2.new(0.46,0,0,19)
fovLabel.BackgroundTransparency = 1
fovLabel.TextColor3 = Color3.fromRGB(255,140,140)
fovLabel.Font = Enum.Font.GothamBlack
fovLabel.TextSize = 15
fovLabel.Text = "FOV: "..fov
fovLabel.ZIndex=2

local btnFovPlus  = Instance.new("TextButton", frame)
btnFovPlus.Size = UDim2.new(0.14,0,0,17)
btnFovPlus.Position = UDim2.new(0.59,0,0,130)
btnFovPlus.BackgroundColor3 = Color3.fromRGB(255,140,140)
btnFovPlus.Text = "+"
btnFovPlus.TextColor3 = Color3.fromRGB(0,0,0)
btnFovPlus.TextSize = 13
btnFovPlus.Font = Enum.Font.GothamBold
btnFovPlus.BorderSizePixel = 0
btnFovPlus.ZIndex=2 round(btnFovPlus,5)
btnFovPlus.MouseButton1Click:Connect(function()
    fov = math.clamp(fov+10,20,400) fovLabel.Text = "FOV: "..fov if fovCircle then fovCircle.Radius = fov end
end)
local btnFovMin   = Instance.new("TextButton", frame)
btnFovMin.Size = UDim2.new(0.14,0,0,17)
btnFovMin.Position = UDim2.new(0.75,0,0,130)
btnFovMin.BackgroundColor3 = Color3.fromRGB(255,140,140)
btnFovMin.Text = "-"
btnFovMin.TextColor3 = Color3.fromRGB(0,0,0)
btnFovMin.TextSize = 13
btnFovMin.Font = Enum.Font.GothamBold
btnFovMin.BorderSizePixel = 0
btnFovMin.ZIndex=2 round(btnFovMin,5)
btnFovMin.MouseButton1Click:Connect(function()
    fov = math.clamp(fov-10,20,400) fovLabel.Text = "FOV: "..fov if fovCircle then fovCircle.Radius = fov end
end)

local resetBtn = Instance.new("TextButton", frame)
resetBtn.Name = "resetBtn"
resetBtn.Size = UDim2.new(0.86,0,0,18)
resetBtn.Position = UDim2.new(0.07,0,0,157)
resetBtn.BackgroundColor3 = Color3.fromRGB(130,60,60)
resetBtn.Text = "Resetar Painel"
resetBtn.TextColor3 = Color3.fromRGB(245,220,220)
resetBtn.TextSize = 11
resetBtn.Font = Enum.Font.GothamBold
resetBtn.Parent = frame
resetBtn.ZIndex=2 round(resetBtn,6)
resetBtn.MouseButton1Click:Connect(function() frame.Position=UDim2.new(0,30,0,120) end)

local infoLabel = Instance.new("TextLabel", frame)
infoLabel.Size = UDim2.new(1,0,0,40)
infoLabel.Position = UDim2.new(0,0,0,186)
infoLabel.Text = "AIM FULL = instantâneo  |  AIM LEGIT = suave\nSegure botão direito do mouse!\nVermelho: ESP & FOV"
infoLabel.TextColor3 = Color3.fromRGB(245,130,130)
infoLabel.BackgroundTransparency = 1
infoLabel.Font = Enum.Font.Gotham
infoLabel.TextSize = 10
infoLabel.TextWrapped = true
infoLabel.ZIndex=2

----------------------------------------
-- FOV CÍRCULO (Vermelho claro)
local fovCircle = Drawing and Drawing.new("Circle") or nil
if fovCircle then
    fovCircle.Radius = fov
    fovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    fovCircle.Color = fovColor
    fovCircle.Thickness = 2
    fovCircle.Filled = false
    fovCircle.Visible = false
end

RunService.RenderStepped:Connect(function()
    if fovCircle then
        fovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
        fovCircle.Radius = fov
        fovCircle.Color = fovColor
        fovCircle.Visible = showFovCircle
    end
end)

----------------------------------------
-- AIMBOT (FULL / LEGIT)
local currentTarget = nil
local function getClosestTarget()
    local closest, minDist = nil, math.huge
    if not Camera then return nil end
    local vp = Camera.ViewportSize
    local screenCenter = Vector2.new(vp.X/2, vp.Y/2)
    local playerPosition = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character.HumanoidRootPart.Position or Vector3.new(0,0,0)
    for _,plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            local part = headAimEnabled and plr.Character:FindFirstChild("Head") or plr.Character:FindFirstChild("HumanoidRootPart")
            local hum = plr.Character:FindFirstChild("Humanoid")
            if part and hum and hum.Health > 0 then
                local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
                local dist = (Vector2.new(pos.X,pos.Y)-screenCenter).Magnitude
                if onScreen and dist < minDist and dist <= fov then
                    if not teamCheck or plr.Team ~= LocalPlayer.Team then
                        closest,minDist = plr,dist
                    end
                end
            end
        end
    end
    return closest
end

RunService.RenderStepped:Connect(function()
    if (aimMode=="FULL" or aimMode=="LEGIT") and UserInputService:IsMouseButtonPressed(aimKey) then
        if not currentTarget then
            currentTarget = getClosestTarget()
        end
        if currentTarget and currentTarget.Character then
            local part = headAimEnabled and currentTarget.Character:FindFirstChild("Head") or currentTarget.Character:FindFirstChild("HumanoidRootPart")
            if part then
                if aimMode=="FULL" then
                    Camera.CFrame = CFrame.new(Camera.CFrame.Position, part.Position)
                else
                    Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, part.Position), legitSmooth)
                end
            end
        end
    else
        currentTarget = nil
    end
end)

----------------------------------------
-- ESP - Highlight vermelho com respawn
local highlights = {}
local function removeHighlight(player)
    if highlights[player] then highlights[player]:Destroy() highlights[player]=nil end
end
local function setupHighlightForCharacter(player,character)
    removeHighlight(player)
    if character:FindFirstChild("HumanoidRootPart") then
        local h=Instance.new("Highlight")
        h.Adornee = character
        h.FillColor = espColor
        h.FillTransparency = 0.22
        h.OutlineColor = outlineESP
        h.OutlineTransparency = 0.25
        h.Enabled = espEnabled
        h.Parent = character
        highlights[player]=h
    end
end
local function handlePlayer(player)
    if player==LocalPlayer then return end
    player.CharacterAdded:Connect(function(char)
        repeat task.wait() until char:FindFirstChild("HumanoidRootPart")
        setupHighlightForCharacter(player,char)
    end)
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        setupHighlightForCharacter(player,player.Character)
    end
end

Players.PlayerRemoving:Connect(function(plr) removeHighlight(plr) end)
for _,player in ipairs(Players:GetPlayers()) do handlePlayer(player) end
Players.PlayerAdded:Connect(handlePlayer)

RunService.RenderStepped:Connect(function()
    for plr,h in pairs(highlights) do
        if h and h.Parent then h.Enabled=espEnabled end
    end
end)
