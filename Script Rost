-- AimBot with Wall Check & ESP
local AimSettings = {
    Enabled = true,
    TeamCheck = true,
    WallCheck = true,
    FOV = 100,
    Smoothness = 0.2,
    AimKey = Enum.UserInputType.MouseButton2,
    AimPart = "Head"
}

local ESPSettings = {
    Enabled = true,
    Boxes = true,
    Names = true,
    Health = true,
    Distance = true
}

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- ESP Functions
local function CreateESP(player)
    local esp = {
        Box = Drawing.new("Quad"),
        Name = Drawing.new("Text"),
        Health = Drawing.new("Text"),
        Distance = Drawing.new("Text")
    }
    
    local function UpdateESP()
        if not player.Character or not player.Character:FindFirstChild("Humanoid") or not player.Character:FindFirstChild("HumanoidRootPart") then
            return false
        end
        
        local rootPart = player.Character.HumanoidRootPart
        local head = player.Character:FindFirstChild("Head")
        if not head then return false end
        
        local position, onScreen = Camera:WorldToViewportPoint(rootPart.Position)
        if not onScreen then return false end
        
        local scale = 1 / (position.Z * math.tan(math.rad(Camera.FieldOfView / 2)) * 2) * 1000
        local width, height = 4 * scale, 5 * scale
        local x, y = position.X, position.Y
        
        -- Box ESP
        if ESPSettings.Boxes then
            esp.Box.Visible = true
            esp.Box.PointA = Vector2.new(x - width/2, y - height/2)
            esp.Box.PointB = Vector2.new(x + width/2, y - height/2)
            esp.Box.PointC = Vector2.new(x + width/2, y + height/2)
            esp.Box.PointD = Vector2.new(x - width/2, y + height/2)
            esp.Box.Color = Color3.fromRGB(255, 255, 255)
            esp.Box.Thickness = 1
        else
            esp.Box.Visible = false
        end
        
        -- Name ESP
        if ESPSettings.Names then
            esp.Name.Visible = true
            esp.Name.Text = player.Name
            esp.Name.Position = Vector2.new(x, y - height/2 - 15)
            esp.Name.Color = Color3.fromRGB(255, 255, 255)
            esp.Name.Size = 13
            esp.Name.Center = true
        else
            esp.Name.Visible = false
        end
        
        -- Health ESP
        if ESPSettings.Health then
            esp.Health.Visible = true
            esp.Health.Text = "HP: "..math.floor(player.Character.Humanoid.Health)
            esp.Health.Position = Vector2.new(x, y + height/2 + 5)
            esp.Health.Color = Color3.fromRGB(0, 255, 0)
            esp.Health.Size = 13
            esp.Health.Center = true
        else
            esp.Health.Visible = false
        end
        
        -- Distance ESP
        if ESPSettings.Distance then
            esp.Distance.Visible = true
            local distance = (rootPart.Position - Camera.CFrame.Position).Magnitude
            esp.Distance.Text = math.floor(distance).."m"
            esp.Distance.Position = Vector2.new(x, y + height/2 + 20)
            esp.Distance.Color = Color3.fromRGB(255, 255, 0)
            esp.Distance.Size = 13
            esp.Distance.Center = true
        else
            esp.Distance.Visible = false
        end
        
        return true
    end
    
    coroutine.wrap(function()
        while player and player.Parent do
            if not UpdateESP() then break end
            RunService.RenderStepped:Wait()
        end
        
        -- Clean up
        for _, drawing in pairs(esp) do
            drawing:Remove()
        end
    end)()
end

-- Wall Check
local function IsVisible(part)
    if not AimSettings.WallCheck then return true end
    
    local origin = Camera.CFrame.Position
    local target = part.Position
    local direction = (target - origin).Unit * (origin - target).Magnitude
    
    local raycastParams = RaycastParams.new()
    raycastParams.FilterDescendantsInstances = {LocalPlayer.Character, part.Parent}
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    
    local raycastResult = workspace:Raycast(origin, direction, raycastParams)
    
    if raycastResult then
        return raycastResult.Instance:IsDescendantOf(part.Parent)
    end
    
    return true
end

-- AimBot Functions
local function GetClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = AimSettings.FOV
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if AimSettings.TeamCheck and player.Team == LocalPlayer.Team then continue end
            
            local rootPart = player.Character.HumanoidRootPart
            local screenPoint = Camera:WorldToViewportPoint(rootPart.Position)
            local magnitude = (Vector2.new(screenPoint.X, screenPoint.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
            
            if magnitude < shortestDistance and IsVisible(rootPart) then
                closestPlayer = player
                shortestDistance = magnitude
            end
        end
    end
    
    return closestPlayer
end

-- Main Loop
RunService.RenderStepped:Connect(function()
    if not AimSettings.Enabled or not UserInputService:IsMouseButtonPressed(AimSettings.AimKey) then return end
    
    local target = GetClosestPlayer()
    if not target or not target.Character or not target.Character:FindFirstChild(AimSettings.AimPart) then return end
    
    local targetPart = target.Character[AimSettings.AimPart]
    local targetPosition = targetPart.Position + (targetPart.Velocity * 0.15) -- Prediction
    
    local currentCamera = Camera.CFrame
    local newCamera = CFrame.new(currentCamera.Position, targetPosition)
    
    Camera.CFrame = currentCamera:Lerp(newCamera, AimSettings.Smoothness)
end)

-- ESP Initialization
for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        CreateESP(player)
    end
end

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        CreateESP(player)
    end)
end)

-- Simple UI
local function CreateUI()
    local library = loadstring(game:HttpGet("https://raw.githubusercontent.com/wally-rblx/uwuware-ui/main/main.lua"))()
    local window = library:CreateWindow("AimBot v3")
    
    local aimTab = window:AddFolder("AimBot")
    aimTab:AddToggle({text = "Enabled", flag = "aim_enabled", callback = function(val) AimSettings.Enabled = val end})
    aimTab:AddToggle({text = "Team Check", flag = "aim_teamcheck", callback = function(val) AimSettings.TeamCheck = val end})
    aimTab:AddToggle({text = "Wall Check", flag = "aim_wallcheck", callback = function(val) AimSettings.WallCheck = val end})
    aimTab:AddSlider({text = "FOV", min = 1, max = 360, value = 100, callback = function(val) AimSettings.FOV = val end})
    aimTab:AddSlider({text = "Smoothness", min = 0, max = 1, float = 0.1, value = 0.2, callback = function(val) AimSettings.Smoothness = val end})
    aimTab:AddList({text = "Aim Part", values = {"Head", "HumanoidRootPart"}, callback = function(val) AimSettings.AimPart = val end})
    
    local espTab = window:AddFolder("ESP")
    espTab:AddToggle({text = "Enabled", flag = "esp_enabled", callback = function(val) ESPSettings.Enabled = val end})
    espTab:AddToggle({text = "Boxes", flag = "esp_boxes", callback = function(val) ESPSettings.Boxes = val end})
    espTab:AddToggle({text = "Names", flag = "esp_names", callback = function(val) ESPSettings.Names = val end})
    espTab:AddToggle({text = "Health", flag = "esp_health", callback = function(val) ESPSettings.Health = val end})
    espTab:AddToggle({text = "Distance", flag = "esp_distance", callback = function(val) ESPSettings.Distance = val end})
    
    library:Init()
end

CreateUI()
