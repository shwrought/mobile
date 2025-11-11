-- ========================================
-- AUTO RECRUIT SOLO 150 RZ$ (PERÚ - 100% FUNCIONANDO 11 NOV 2025)
-- SOLO COMPRA LOS QUE CUESTAN EXACTAMENTE "150 RZ$"
-- ========================================

getgenv().AutoRecruit150RZ = true  -- true = ON | false = OFF

local Players = game:GetService("Players")
local PathfindingService = game:GetService("PathfindingService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootpart = character:WaitForChild("HumanoidRootPart")

spawn(function()
    while wait(0.6) do
        if not getgenv().AutoRecruit150RZ then
            wait(2)
            continue
        end

        local encontrado = false

        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("ProximityPrompt") then
                local texto = tostring(obj.ObjectText)

                -- DETECCIÓN ULTRA PRECISA: SOLO "150 RZ$"
                if texto == "150 RZ$" or texto:find("150 RZ$") and texto:gsub("150 RZ$", "") == "" then
                    encontrado = true
                    
                    local modelo = obj:FindFirstAncestorWhichIsA("Model")
                    if modelo and modelo:FindFirstChild("HumanoidRootPart") then
                        local hrp = modelo.HumanoidRootPart
                        local distancia = (rootpart.Position - hrp.Position).Magnitude

                        print("150 RZ$ DETECTADO → " .. modelo.Name .. " | " .. math.floor(distancia) .. " studs")

                        -- Caminar normal si está lejos
                        if distancia > 16 then
                            humanoid:MoveTo(hrp.Position + Vector3.new(0,0,-4))
                            humanoid.MoveToFinished:Wait(10)
                        end

                        -- LOOP INFINITO HASTA QUE LO COMPRE
                        repeat
                            if not getgenv().AutoRecruit150RZ then break end
                            
                            pcall(function()
                                obj.HoldDuration = 0
                                obj.MaxActivationDistance = 40
                                obj.RequiresLineOfSight = false
                                fireproximityprompt(obj)
                            end)
                            
                            wait(0.35)
                        until not obj.Parent or not obj:IsDescendantOf(workspace)

                        print("¡RECLUTADO 150 RZ$ CON ÉXITO! → " .. modelo.Name)
                        wait(1.8)
                    end
                end
            end
        end

        if not encontrado then
            print("Esperando nuevo brainrot de 150 RZ$...")
        end
    end
end)

-- Respawn (nunca se rompe)
player.CharacterAdded:Connect(function(newChar)
    character = newChar
    humanoid = character:WaitForChild("Humanoid")
    rootpart = character:WaitForChild("HumanoidRootPart")
end)

print("AUTO 150 RZ$ ACTIVADO (PERÚ - 11 NOV 2025)")
print("SOLO COMPRA LOS DE 150 RZ$")
print("PAUSAR → getgenv().AutoRecruit150RZ = false")
print("REANUDAR → getgenv().AutoRecruit150RZ = true")
