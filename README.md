loadstring([==[
-- ===============================
-- VMS HUB - LEANDRO (CATHUB / DELTA)
-- ===============================
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local VirtualUser = game:GetService("VirtualUser")

-- cria GUI
local HubUI = Instance.new("ScreenGui")
HubUI.Name = "VMSHub"
HubUI.ResetOnSpawn = false
HubUI.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- ============= TELA DE LOGIN =============
local LoginFrame = Instance.new("Frame")
LoginFrame.Size = UDim2.new(0,400,0,300)
LoginFrame.Position = UDim2.new(0.5,-200,0.5,-150)
LoginFrame.BackgroundColor3 = Color3.fromRGB(20,20,20)
LoginFrame.Parent = HubUI

local PasswordBox = Instance.new("TextBox")
PasswordBox.Size = UDim2.new(0,200,0,40)
PasswordBox.Position = UDim2.new(0.5,-100,0.5,-20)
PasswordBox.PlaceholderText = "Digite a senha"
PasswordBox.TextScaled = true
PasswordBox.ClearTextOnFocus = true
PasswordBox.Parent = LoginFrame

local LoginButton = Instance.new("TextButton")
LoginButton.Size = UDim2.new(0,100,0,40)
LoginButton.Position = UDim2.new(0.5,-50,0.7,0)
LoginButton.Text = "Login"
LoginButton.Parent = LoginFrame

-- ============= HUB PRINCIPAL =============
local HubFrame = Instance.new("Frame")
HubFrame.Size = UDim2.new(0,500,0,500)
HubFrame.Position = UDim2.new(0.5,-250,0.5,-250)
HubFrame.BackgroundColor3 = Color3.fromRGB(25,25,25)
HubFrame.Parent = HubUI
HubFrame.Visible = false

-- Título / Criador
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1,0,0,50)
Title.Text = "VMS Hub"
Title.TextScaled = true
Title.TextColor3 = Color3.fromRGB(255,255,255)
Title.Font = Enum.Font.GothamBold
Title.BackgroundTransparency = 1
Title.Parent = HubFrame

local CreatorLabel = Instance.new("TextLabel")
CreatorLabel.Size = UDim2.new(0,150,0,30)
CreatorLabel.Position = UDim2.new(0,10,0,10)
CreatorLabel.Text = "Criador: Leandro"
CreatorLabel.TextScaled = true
CreatorLabel.BackgroundTransparency = 1
CreatorLabel.TextColor3 = Color3.fromRGB(255,255,255)
CreatorLabel.Parent = HubFrame

-- IMAGEM DO JOGADOR (ALONGADO)
local PlayerImageFrame = Instance.new("Frame")
PlayerImageFrame.Size = UDim2.new(0,60,0,40) -- alongado
PlayerImageFrame.Position = UDim2.new(1,-70,0,10)
PlayerImageFrame.BackgroundColor3 = Color3.fromRGB(50,50,50)
PlayerImageFrame.ClipsDescendants = true
PlayerImageFrame.Parent = HubFrame
local UICorner = Instance.new("UICorner", PlayerImageFrame)
UICorner.CornerRadius = UDim.new(1,0)

local PlayerImage = Instance.new("ImageLabel")
PlayerImage.Size = UDim2.new(1,0,1,0)
PlayerImage.BackgroundTransparency = 1
PlayerImage.Parent = PlayerImageFrame
local success, thumbUrl = pcall(function()
    return Players:GetUserThumbnailAsync(LocalPlayer.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size420x420)
end)
if success then
    PlayerImage.Image = thumbUrl
end

local MinimizeButton = Instance.new("TextButton")
MinimizeButton.Size = UDim2.new(0,50,0,30)
MinimizeButton.Position = UDim2.new(1,-60,0,10)
MinimizeButton.Text = "_"
MinimizeButton.Parent = HubFrame
MinimizeButton.MouseButton1Click:Connect(function() HubFrame.Visible = false end)

LoginButton.MouseButton1Click:Connect(function()
    if PasswordBox.Text == "Leandro" then
        LoginFrame.Visible = false
        HubFrame.Visible = true
    else
        PasswordBox.Text = ""
        PasswordBox.PlaceholderText = "Senha incorreta!"
    end
end)

-- ================= BLOCO IMBOT =================
local IMBlock = Instance.new("Frame")
IMBlock.Size = UDim2.new(0,480,0,220)
IMBlock.Position = UDim2.new(0,10,0,60)
IMBlock.BackgroundColor3 = Color3.fromRGB(35,35,35)
IMBlock.Parent = HubFrame

local IMTitle = Instance.new("TextLabel")
IMTitle.Size = UDim2.new(1,0,0,28)
IMTitle.BackgroundTransparency = 1
IMTitle.Text = "IMBOT (Bloco Único)"
IMTitle.TextColor3 = Color3.fromRGB(255,255,255)
IMTitle.Font = Enum.Font.GothamBold
IMTitle.TextScaled = true
IMTitle.Parent = IMBlock

local FOVEnabled, FOVSize, FOVColor = false, 100, Color3.fromRGB(0,255,0)
local IMShotEnabled, ESPEnabled, IMWalkSpeed = false, false, 16

local DrawingAvailable, Drawing = pcall(function() return Drawing end)
if not DrawingAvailable then Drawing = nil end
local FOVCircle = nil
if Drawing then
    FOVCircle = Drawing.new("Circle")
    FOVCircle.Visible = false
    FOVCircle.Radius = FOVSize
    FOVCircle.Color = FOVColor
    FOVCircle.Thickness = 2
    FOVCircle.Filled = false
    FOVCircle.Transparency = 1
end
local ESPLines = {}

local function fireAtHead(targetPlayer)
    if not (targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("Head")) then return end
    pcall(function()
        local headPos = targetPlayer.Character.Head.Position
        VirtualUser:CaptureController()
        VirtualUser:Button1Down(Vector2.new(0,0))
        task.wait(0.02)
        VirtualUser:Button1Up(Vector2.new(0,0))
    end)
end

local function IMShotLoop()
    if not (FOVEnabled and IMShotEnabled) then return end
    local screenCenter = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    for _,pl in pairs(Players:GetPlayers()) do
        if pl ~= LocalPlayer and pl.Character and pl.Character:FindFirstChild("Head") and pl.Character:FindFirstChild("Humanoid") and pl.Character.Humanoid.Health > 0 then
            local headPos, onScreen = Camera:WorldToViewportPoint(pl.Character.Head.Position)
            if onScreen then
                local dist = (Vector2.new(headPos.X, headPos.Y) - screenCenter).Magnitude
                if dist <= FOVSize then fireAtHead(pl) end
            end
        end
    end
end

local function ClearESP()
    for _,v in pairs(ESPLines) do if v then pcall(function() v:Remove() end) end end
    ESPLines = {}
end

local function UpdateESP()
    if not Drawing then return end
    ClearESP()
    for _,pl in pairs(Players:GetPlayers()) do
        if pl ~= LocalPlayer and pl.Character and pl.Character:FindFirstChild("Head") and pl.Character:FindFirstChild("HumanoidRootPart") then
            local headPos, headOn = Camera:WorldToViewportPoint(pl.Character.Head.Position)
            local rootPos, rootOn = Camera:WorldToViewportPoint(pl.Character.HumanoidRootPart.Position)
            if headOn and rootOn then
                local head2 = Vector2.new(headPos.X, headPos.Y)
                local root2 = Vector2.new(rootPos.X, rootPos.Y)
                local line = Drawing.new("Line")
                line.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
                line.To = head2
                line.Thickness = 1
                line.Transparency = 1
                line.Color = FOVColor
                table.insert(ESPLines, line)
                local skel = Drawing.new("Line")
                skel.From = head2
                skel.To = root2
                skel.Thickness = 1
                skel.Transparency = 1
                skel.Color = Color3.fromRGB(255,255,255)
                table.insert(ESPLines, skel)
            end
        end
    end
end

RunService.RenderStepped:Connect(function()
    if Drawing and FOVCircle then
        FOVCircle.Visible = FOVEnabled
        FOVCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
        FOVCircle.Radius = FOVSize
        FOVCircle.Color = FOVColor
    end
    if IMShotEnabled and FOVEnabled then IMShotLoop() end
    if ESPEnabled and Drawing then UpdateESP() else if Drawing then ClearESP() end end
end)

-- CONTROLES IMBOT
local FOVToggle = Instance.new("TextButton")
FOVToggle.Size = UDim2.new(0,110,0,34)
FOVToggle.Position = UDim2.new(0,8,0,38)
FOVToggle.BackgroundColor3 = Color3.fromRGB(50,50,50)
FOVToggle.TextColor3 = Color3.fromRGB(255,255,255)
FOVToggle.Text = "FOV OFF"
FOVToggle.Parent = IMBlock
FOVToggle.MouseButton1Click:Connect(function()
    FOVEnabled = not FOVEnabled
    IMShotEnabled = FOVEnabled
    FOVToggle.Text = "FOV "..(FOVEnabled and "ON" or "OFF")
end)

local IMShotToggle = Instance.new("TextButton")
IMShotToggle.Size = UDim2.new(0,110,0,34)
IMShotToggle.Position = UDim2.new(0,130,0,38)
IMShotToggle.BackgroundColor3 = Color3.fromRGB(50,50,50)
IMShotToggle.TextColor3 = Color3.fromRGB(255,255,255)
IMShotToggle.Text = "IMShot OFF"
IMShotToggle.Parent = IMBlock
IMShotToggle.MouseButton1Click:Connect(function# Leandro-scripts-
