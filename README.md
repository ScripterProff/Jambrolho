-- Jamba Hub UI com sistema de abas, ESP e janela movível com botão de fechar

local Players = game:GetService("Players") local LocalPlayer = Players.LocalPlayer local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

if PlayerGui:FindFirstChild("JambaHubUI") then PlayerGui:FindFirstChild("JambaHubUI"):Destroy() end

local ScreenGui = Instance.new("ScreenGui") ScreenGui.Name = "JambaHubUI" ScreenGui.ResetOnSpawn = false ScreenGui.Parent = PlayerGui

local MainFrame = Instance.new("Frame") MainFrame.Size = UDim2.new(0, 500, 0, 300) MainFrame.Position = UDim2.new(0.5, -250, 0.5, -150) MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25) MainFrame.BorderSizePixel = 0 MainFrame.Active = true MainFrame.Draggable = true MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner") UICorner.CornerRadius = UDim.new(0, 8) UICorner.Parent = MainFrame

local SideMenu = Instance.new("Frame") SideMenu.Size = UDim2.new(0, 120, 1, 0) SideMenu.Position = UDim2.new(0, 0, 0, 0) SideMenu.BackgroundColor3 = Color3.fromRGB(30, 30, 30) SideMenu.Parent = MainFrame

local Title = Instance.new("TextLabel") Title.Size = UDim2.new(1, -30, 0, 40) Title.Position = UDim2.new(0, 0, 0, 0) Title.BackgroundTransparency = 1 Title.Text = "  Jamba Hub" Title.TextColor3 = Color3.new(1, 1, 1) Title.Font = Enum.Font.SourceSansBold Title.TextSize = 20 Title.TextXAlignment = Enum.TextXAlignment.Left Title.Parent = SideMenu

-- Botão de fechar local CloseButton = Instance.new("TextButton") CloseButton.Size = UDim2.new(0, 30, 0, 30) CloseButton.Position = UDim2.new(1, -35, 0, 5) CloseButton.Text = "X" CloseButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50) CloseButton.TextColor3 = Color3.new(1, 1, 1) CloseButton.Font = Enum.Font.SourceSansBold CloseButton.TextSize = 18 CloseButton.Parent = MainFrame

CloseButton.MouseButton1Click:Connect(function() ScreenGui:Destroy() end)

local function createTabButton(name, posY) local button = Instance.new("TextButton") button.Size = UDim2.new(1, 0, 0, 40) button.Position = UDim2.new(0, 0, 0, posY) button.Text = name button.BackgroundColor3 = Color3.fromRGB(40, 40, 40) button.TextColor3 = Color3.new(1, 1, 1) button.Parent = SideMenu return button end

local function createTabFrame() local frame = Instance.new("Frame") frame.Size = UDim2.new(1, -120, 1, 0) frame.Position = UDim2.new(0, 120, 0, 0) frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20) frame.Visible = false frame.Parent = MainFrame return frame end

local tabs = {} local tabNames = {"MAIN", "PLAYER", "VISUALS", "ANIMATIONS", "EMOTE", "SUS", "TROLLS"} local activeTab = nil

for i, name in ipairs(tabNames) do local button = createTabButton(name, 45 + (i - 1) * 45) local frame = createTabFrame() tabs[name] = frame

button.MouseButton1Click:Connect(function()
    if activeTab then tabs[activeTab].Visible = false end
    frame.Visible = true
    activeTab = name
end)

end

local VisualsFrame = tabs["VISUALS"] local ESPToggle = Instance.new("TextButton", VisualsFrame) ESPToggle.Size = UDim2.new(0, 200, 0, 40) ESPToggle.Position = UDim2.new(0, 20, 0, 20) ESPToggle.Text = "ESP OFF" ESPToggle.BackgroundColor3 = Color3.fromRGB(50, 50, 50) ESPToggle.TextColor3 = Color3.new(1, 1, 1)

local RunService = game:GetService("RunService") local Camera = workspace.CurrentCamera local DrawingESP = false local ESPObjects = {}

local function createDrawing(type, props) local obj = Drawing.new(type) for i, v in pairs(props) do obj[i] = v end return obj end

local function clearESP(player) if ESPObjects[player] then for _, obj in pairs(ESPObjects[player]) do obj:Remove() end ESPObjects[player] = nil end end

local function updateESP(player) if player == LocalPlayer then return end local char = player.Character if not char then return end local head = char:FindFirstChild("Head") local hrp = char:FindFirstChild("HumanoidRootPart") if not head or not hrp then return end

local headPos, onScreen = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.3, 0))
local footPos = Camera:WorldToViewportPoint(hrp.Position - Vector3.new(0, 2.5, 0))
if not onScreen then clearESP(player) return end

local height = footPos.Y - headPos.Y
local width = height / 2.2
local tl = Vector2.new(headPos.X - width / 2, headPos.Y)
local tr = Vector2.new(headPos.X + width / 2, headPos.Y)
local bl = Vector2.new(footPos.X - width / 2, footPos.Y)
local br = Vector2.new(footPos.X + width / 2, footPos.Y)

if not ESPObjects[player] then
    ESPObjects[player] = {
        Top = createDrawing("Line", {Thickness = 1.5, Color = Color3.new(1,0,0), Transparency = 1}),
        Bottom = createDrawing("Line", {Thickness = 1.5, Color = Color3.new(1,0,0), Transparency = 1}),
        Left = createDrawing("Line", {Thickness = 1.5, Color = Color3.new(1,0,0), Transparency = 1}),
        Right = createDrawing("Line", {Thickness = 1.5, Color = Color3.new(1,0,0), Transparency = 1}),
        Tracer = createDrawing("Line", {Thickness = 1.5, Color = Color3.new(1,0,0), Transparency = 1})
    }
end

local box = ESPObjects[player]
box.Top.From = tl box.Top.To = tr
box.Bottom.From = bl box.Bottom.To = br
box.Left.From = tl box.Left.To = bl
box.Right.From = tr box.Right.To = br
box.Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
box.Tracer.To = bl

for _, obj in pairs(box) do obj.Visible = true end

end

RunService.RenderStepped:Connect(function() if not DrawingESP then for _, p in pairs(Players:GetPlayers()) do clearESP(p) end return end for _, p in pairs(Players:GetPlayers()) do updateESP(p) end end)

ESPToggle.MouseButton1Click:Connect(function() DrawingESP = not DrawingESP ESPToggle.Text = DrawingESP and "ESP ON" or "ESP OFF" end)

