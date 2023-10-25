# gas-casino
mini sports book with GAS theme

# About

Junior CS student | Exploring WΞB3 (Solidity)/NFTs



combatant.h
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Character.h"
#include "Combatant.generated.h"

UCLASS()
class DARKSOULS_BOSS_FIGHT_API ACombatant : public ACharacter
{
	GENERATED_BODY()

public:
	// Sets default values for this character's properties
	ACombatant();

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	// Called to bind functionality to input
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

	AActor* Target;

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Combat")
		bool TargetLocked;

	bool Attacking;
	bool AttackDamaging;
	bool MovingForward;
	bool MovingBackwards;
	bool NextAttackReady;
	bool Stumbling;
	
	bool RotateTowardsTarget;

	UPROPERTY(EditAnywhere, Category = "Animation")
		float RotationSmoothing;

	UPROPERTY(EditAnywhere, Category = "Animations")
		TArray<UAnimMontage*> AttackAnimations;

	UPROPERTY(EditAnywhere, Category = "Animations")
		TArray<UAnimMontage*> TakeHit_StumbleBackwards;

	// Actors hit with the last attack - Used to stop duplicate hits
	TArray<AActor*> AttackHitActors;

	virtual void Attack();

	// anim called: rotate and jump towards target
	UFUNCTION(BlueprintCallable, Category = "Combat")
		virtual void AttackLunge();

	// anim called: rotate and jump towards target
	UFUNCTION(BlueprintCallable, Category = "Combat")
		virtual void EndAttack();

	// set if weapon applies damage
	UFUNCTION(BlueprintCallable, Category = "Combat")
		virtual void SetAttackDamaging(bool Damaging);

	// anim called: set if moving forward
	UFUNCTION(BlueprintCallable, Category = "Animation")
		virtual void SetMovingForward(bool IsMovingForward);

	// anim called: set if moving backwards
	UFUNCTION(BlueprintCallable, Category = "Animation")
		virtual void SetMovingBackwards(bool IsMovingBackwards);

	// anim called: set if moving backwards
	UFUNCTION(BlueprintCallable, Category = "Animation")
		virtual void EndStumble();

	// called by anim to singlat that the next attack is potentially allowed
	UFUNCTION(BlueprintCallable, Category = "Combat")
		virtual void AttackNextReady();

	virtual void LookAtSmooth();

	// anim called: get rate of actors look rotation
	UFUNCTION(BlueprintCallable, Category = "Animation")
		float GetCurrentRotationSpeed();
	
	float LastRotationSpeed;

};











// Fill out your copyright notice in the Description page of Project Settings.

#include "Combatant.h"
#include "GameFramework/CharacterMovementComponent.h"

// Sets default values
ACombatant::ACombatant()
{
	// Set this character to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	TargetLocked = false;
	NextAttackReady = false;
	Attacking = false;
	AttackDamaging = false;
	MovingForward = false;
	MovingBackwards = false;
	RotateTowardsTarget = true;
	Stumbling = false;
	RotationSmoothing = 5.0f;
	LastRotationSpeed = 0.0f;

}

// Called when the game starts or when spawned
void ACombatant::BeginPlay()
{
	Super::BeginPlay();
	
}

// Called every frame
void ACombatant::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	if(RotateTowardsTarget)
		LookAtSmooth();
	
}

// Called to bind functionality to input
void ACombatant::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

}

void ACombatant::Attack()
{
	Attacking = true;
	NextAttackReady = false;
	AttackDamaging = false;

	//empty atrray
	AttackHitActors.Empty();
}

void ACombatant::AttackLunge()
{
	if(Target != NULL)
	{
		FVector Direction = Target->GetActorLocation() - GetActorLocation();
		Direction = FVector(Direction.X, Direction.Y, 0);
		FRotator Rotation = FRotationMatrix::MakeFromX(Direction).Rotator();
		SetActorRotation(Rotation);
	}

	FVector NewLocation = GetActorLocation() + (GetActorForwardVector() * 70);
	SetActorLocation(NewLocation, true);
}

void ACombatant::EndAttack()
{
	Attacking = false;
	NextAttackReady = false;
}

void ACombatant::SetAttackDamaging(bool Damaging)
{
	AttackDamaging = Damaging;
}

void ACombatant::SetMovingForward(bool IsMovingForward)
{
	MovingForward = IsMovingForward;
}

void ACombatant::SetMovingBackwards(bool IsMovingBackwards)
{
	MovingBackwards = IsMovingBackwards;
}

void ACombatant::EndStumble()
{
	Stumbling = false;
}

void ACombatant::AttackNextReady()
{
	NextAttackReady = true;
}

void ACombatant::LookAtSmooth()
{
	if(Target != NULL && TargetLocked &&      
		!Attacking && !GetCharacterMovement()->IsFalling())
	{
		FVector Direction = Target->GetActorLocation() - GetActorLocation();
		Direction = FVector(Direction.X, Direction.Y, 0);
		FRotator Rotation = FRotationMatrix::MakeFromX(Direction).Rotator();

		FRotator SmoothedRotation = FMath::Lerp(GetActorRotation(), Rotation,
			RotationSmoothing * GetWorld()->DeltaTimeSeconds);

		LastRotationSpeed = SmoothedRotation.Yaw - GetActorRotation().Yaw;

		SetActorRotation(SmoothedRotation);
		
	}
		
}

float ACombatant::GetCurrentRotationSpeed()
{

	if (RotateTowardsTarget)
		return LastRotationSpeed;
	
	return 0.0f;
}


Combatant.cpp
