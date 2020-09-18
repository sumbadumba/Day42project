# Day42project

#include "framework.h"
#include "GameObject.h"


GameObject::GameObject(D2D1_POINT_2F pos, FLOAT rot, D2D1_VECTOR_2F scale,float moveSpeed)
	: GameImage(pos, rot, scale), moveSpeed(moveSpeed)
{
	bound = new RectBoundary();
}

GameObject::~GameObject()
{
	SAFE_DELETE(bound);
}

void GameObject::UpdateDrawRect()
{
	__super::UpdateDrawRect();
	bound->SetBoundary(DrawRect());
}

void GameObject::Move(D2D1_VECTOR_2F delta, vector<RectBoundary*> bounds)
{
	D2D1_VECTOR_2F adjustDelta = CheckBounds(delta, bound, bounds);

	Image::Move(adjustDelta);
}

D2D1_VECTOR_2F GameObject::CheckBounds(D2D1_VECTOR_2F moveDelta, RectBoundary * targetBound, vector<RectBoundary*> bounds)
{
	D2D1_VECTOR_2F adjustDelta;
	adjustDelta.x = adjustDelta.y = 0.0F;

	D2D1_POINT_2F lefttop;
	D2D1_POINT_2F righttop;
	D2D1_POINT_2F leftbottom;
	D2D1_POINT_2F rightbottom;

	lefttop.x = targetBound->Infimum().x + moveDelta.x;
	lefttop.y = targetBound->Infimum().y + moveDelta.y;
	righttop.x = targetBound->Supremum().x + moveDelta.x;
	righttop.y = targetBound->Infimum().y + moveDelta.y;
	leftbottom.x = targetBound->Infimum().x + moveDelta.x;
	leftbottom.y = targetBound->Supremum().y + moveDelta.y;
	rightbottom.x = targetBound->Supremum().x + moveDelta.x;
	rightbottom.y = targetBound->Supremum().y + moveDelta.y;

	D2D1_VECTOR_2F delta;

	bool intersect = false;

	vector<RectBoundary*>::iterator iter = bounds.begin();
	while (iter != bounds.end())
	{
		if ((*iter) == targetBound)
		{
			iter++;
			continue;
		}
		//
		delta.x = delta.y = 0.0F;

		if (RectBoundary::CheckIntersect((*iter), lefttop))
		{
			intersect = true;
			// left
			if (moveDelta.x < 0.0f && lefttop.x <= (*iter)->Supremum().x)
			{
				// 값이 같은 경우도 바운드 체크하므로 매우 작은 수를 더해서 이후에도 Y 축 이동은 가능하도록 함
				delta.x = ((*iter)->Supremum().x - targetBound->Infimum().x + 0.0001f) * D2D::Get()->GetTargetRate().x;
				if (delta.x < adjustDelta.x)
					adjustDelta.x = delta.x;
			}
			// top
			if (moveDelta.y < 0.0f && lefttop.y <= (*iter)->Supremum().y)
			{
				// 값이 같은 경우도 바운드 체크하므로 매우 작은 수를 더해서 이후에도 X 축 이동은 가능하도록 함
				delta.y = ((*iter)->Supremum().y - targetBound->Infimum().y + 0.0001f) * D2D::Get()->GetTargetRate().y;
				if (delta.y < adjustDelta.y)
					adjustDelta.y = delta.y;
			}
		}
		else if (RectBoundary::CheckIntersect((*iter), righttop))
		{
			intersect = true;
			// right
			if (moveDelta.x > 0.0f && righttop.x >= (*iter)->Infimum().x)
			{
				delta.x = ((*iter)->Infimum().x - targetBound->Supremum().x - 0.0001f) * D2D::Get()->GetTargetRate().x;
				if (delta.x > adjustDelta.x)
					adjustDelta.x = delta.x;
			}
			// top
			if (moveDelta.y < 0.0f && righttop.y <= (*iter)->Supremum().y)
			{
				delta.y = ((*iter)->Supremum().y - targetBound->Infimum().y + 0.0001f) * D2D::Get()->GetTargetRate().y;
				if (delta.y < adjustDelta.y)
					adjustDelta.y = delta.y;
			}
		}
		else if (RectBoundary::CheckIntersect((*iter), leftbottom))
		{
			intersect = true;
			// left
			if (moveDelta.x < 0.0f && leftbottom.x <= (*iter)->Supremum().x)
			{
				delta.x = ((*iter)->Supremum().x - targetBound->Infimum().x + 0.0001f) * D2D::Get()->GetTargetRate().x;
				if (delta.x < adjustDelta.x)
					adjustDelta.x = delta.x;
			}
			// bottom
			if (moveDelta.y > 0.0f && leftbottom.y >= (*iter)->Infimum().y)
			{
				delta.y = ((*iter)->Infimum().y - targetBound->Supremum().y - 0.0001f) * D2D::Get()->GetTargetRate().y;
				if (delta.y > adjustDelta.y)
					adjustDelta.y = delta.y;
			}
		}
		else if (RectBoundary::CheckIntersect((*iter), rightbottom))
		{
			intersect = true;
			// right
			if (moveDelta.x > 0.0f && rightbottom.x >= (*iter)->Infimum().x)
			{
				delta.x = ((*iter)->Infimum().x - targetBound->Supremum().x - 0.0001f) * D2D::Get()->GetTargetRate().x;
				if (delta.x > adjustDelta.x)
					adjustDelta.x = delta.x;
			}
			// bottom
			if (moveDelta.y > 0.0f && rightbottom.y >= (*iter)->Infimum().y)
			{
				delta.y = ((*iter)->Infimum().y - targetBound->Supremum().y - 0.0001f) * D2D::Get()->GetTargetRate().y;
				if (delta.y > adjustDelta.y)
					adjustDelta.y = delta.y;
			}
		}
		iter++;
	}

	if (!intersect)
		adjustDelta = moveDelta;

	return adjustDelta;
}

void GameObject::MoveTo(D2D1_POINT_2F targetPos)
{
	D2D1_POINT_2F imagePos = Position();

	targetPosition.x =  targetPos.x;
	targetPosition.y = targetPos.y;

	D2D1_VECTOR_3F vector3;
	vector3.x = targetPos.x - imagePos.x;
	vector3.y = targetPos.y - imagePos.y;
	vector3.z = targetPos.z - imagePos.z;


	float distance = D2D1Vec3Lenght(vector3.x, vector3.y, vector3.z);

	moveDirection.x = vector3.x / distance;
	moveDirection.y = vector3.y / distance;
	needMove = true;

}
void GameObject::AutoMove()
{
	D2D1_VECTOR_2F delta;

	float deltaTime = Time::Delta();
	delta.x = moveDirection.x * moveSpeed * deltaTime;
	delta.y = moveDirection.y * moveSpeed * deltaTime;

	// 거리가 1이하 일때 delta 를 조정하자 
	D2D1_POINT_2F imagePos = Position();


	D2D1_VECTOR_3F vector3;
	vector3.x = targetPos.x - imagePos.x;
	vector3.y = targetPos.y - imagePos.y;
	vector3.z = targetPos.z - imagePos.z;

	float distance = D2D1Vec3Lenght(vector3.x, vector3.y, vector3.z);

	if (distance < 1.0f)
	{
		delta.x = vector3.x;
		delta.y = vector3.y;

		needMove = false;

	}
	/////////////////////////////////////
	//지나치게 되는 경우

}

















#pragma once

#include "GameImage.h"

class GameObject : public GameImage
{
public:
	GameObject(D2D1_POINT_2F pos, FLOAT rot, D2D1_VECTOR_2F scale, float moveSpeed);
	~GameObject();

	void Update() override;


	void UpdateDrawRect() override;

	void Move(D2D1_VECTOR_2F delta, vector<RectBoundary*> bounds);
	//
	D2D1_VECTOR_2F CheckBounds(D2D1_VECTOR_2F moveDelta, RectBoundary * targetBound, vector<RectBoundary*> bounds);

	RectBoundary * Bound() { return bound; }

	void MoveTo(D2D1_POINT_2F targetPos);

	bool NeedMove() { return needMove; }

private:
	void AutoMove();


private:
	class RectBoundary * bound;


	D2D1_VECTOR_2F targetPosition;
	D2D1_VECTOR_2F moveDirection;

	float moveSpeed;

	bool needMove;

	
};











#pragma once
class Image
{
public:
	Image(D2D1_POINT_2F pos, FLOAT rot, D2D1_VECTOR_2F scale);
	~Image();
private:
	static const FLOAT Opacity;

public:

	virtual void Update();
	virtual void UpdateDrawRect();

	virtual void Render();
	void RenderBitmapTarget(ID2D1BitmapRenderTarget * bitmapRenderTarget);
	//
	void Load(wstring filePath);

	D2D1_POINT_2F Position() { return position; }
	void Position(D2D1_POINT_2F val) { position = val; }//

	virtual void Move(D2D1_VECTOR_2F delta);

	FLOAT Rotation();
	void Rotate(FLOAT delta, D2D1_POINT_2F rotPoint = Point2F(-0xFFFF, -0xFFFF));

	D2D1_VECTOR_2F Scale() { return scale; }
	void Scaling(D2D1_VECTOR_2F delta);

	IWICFormatConverter * GetFormatConverter()
	{
		return FormatConverter;
	}

	D2D1_RECT_F DrawRect() { return drawRect; }

	ID2D1Bitmap * GetBitmap()
	{
		return Bitmap;
	}

protected:
	D2D1_POINT_2F position;	// center 위치
	FLOAT rotation;	// 회전값
	D2D1_VECTOR_2F scale;	// 배율


	//그려질 원본 위치의 사각형
	D2D1_RECT_F srcRect;


private:
	// 이미지 파일 해석용 인터페이스
	IWICBitmapDecoder * ImageDecoder = NULL;
	// 프레임단위로 해석할 디코더 인터페이스
	IWICBitmapFrameDecode * WicFrameDecoder = NULL;
	// 이미지형식 변환 인터페이스
	IWICFormatConverter * FormatConverter = NULL;

	// Direct2D 용 비트맵 객체 인터페이스
	ID2D1Bitmap * Bitmap = NULL;
	// 이미지 원본 크기
	D2D1_SIZE_F imageSize;

	Matrix3x2F rotationMatrix;

	// 그려질 위치의 사각형
	D2D1_RECT_F drawRect;
};



